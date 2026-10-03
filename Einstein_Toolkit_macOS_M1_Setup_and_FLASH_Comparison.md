# Einstein Toolkit on macOS (Apple Silicon): Setup Summary, Workflow, and FLASH Comparison

> **System used:** MacBook Air with Apple Silicon (M1, `arm64`)  
> **Einstein Toolkit release:** ET_2026_05  
> **Main goal:** Build a working Einstein Toolkit installation for the numerical-relativity / multimessenger workshop, verify it with the `HelloWorld` example, and understand how to use it coming from a FLASH-code background. [1][2]

---

## 1. What the Einstein Toolkit is

The **Einstein Toolkit (ET)** is an open-source computational framework mainly used for **numerical relativity and relativistic astrophysics**. It provides infrastructure and physics modules for solving Einstein's equations, relativistic hydrodynamics/MHD, mesh refinement, I/O, apparent-horizon finding, gravitational-wave extraction, and related problems. [1][3]

The ET is built on the **Cactus Framework**. Cactus is the underlying framework that organizes the code into modular components called **thorns**. [3]

A **thorn** is conceptually similar to a physics or infrastructure unit in FLASH: one thorn may provide spacetime variables, another may evolve hydrodynamics, another may handle mesh refinement, another may perform I/O, and another may calculate diagnostics. [3]

The ET is therefore not a single monolithic program. It is a collection of interoperating thorns that are compiled into one executable and activated/configured at runtime through a parameter file. [3]

---

## 2. A FLASH-user mental model

If you already use **FLASH**, the following analogy is useful. [4]

| FLASH idea | Rough Einstein Toolkit / Cactus analogue |
|---|---|
| FLASH source tree | `~/Cactus/` |
| FLASH Units | Cactus **thorns** |
| `Config` / compile-time unit selection | Thorn list used when building |
| `flash.par` | Cactus `.par` parameter file |
| `./setup ...` | SimFactory/Cactus configuration + build |
| `flash4` executable | `~/Cactus/exe/cactus_sim` / SimFactory-managed executable |
| FLASH runtime parameters | `thorn::parameter = value` in `.par` |
| PARAMESH / AMReX-style mesh infrastructure | Carpet / CarpetX and related thorns |
| FLASH Hydro/MHD units | Relativistic Hydro/MHD evolution thorns |
| FLASH checkpoint / plot files | ET I/O/checkpoint/output files |
| FLASH analysis scripts | Python/Julia/VisIt/ParaView or ET analysis tools |

This is only an analogy: the internal architecture and numerical methods are different, but it is a useful way to understand the workflow. [1][3][4]

The most important conceptual translation is:

```text
FLASH:
physics units + flash.par + flash4
            ↓
         simulation

Einstein Toolkit:
thorns + .par file + Cactus executable
            ↓
         simulation
```

---

## 3. What was installed on the Mac

The machine was prepared with the standard macOS development stack and scientific libraries required by the Einstein Toolkit. [2]

### Core development tools

- Apple Xcode Command Line Tools
- Homebrew
- Git
- `curl`
- Apple `clang`
- Homebrew GCC 15
- Homebrew `gfortran-15`
- CMake
- GNU/Linux-style development libraries required by ET [2][5]

### Scientific libraries

- Open MPI 5.x
- HDF5
- Boost
- `libaec` / `libsz`
- hwloc
- other libraries built automatically by the Einstein Toolkit dependency system as required [2][5]

Python was already available through Homebrew, while an Anaconda installation was also present on the machine. During the ET build, Conda was deactivated to avoid compiler and environment conflicts. This was a local machine-specific issue from this setup session.

---

## 4. Why Conda and MESA caused problems

The original shell environment contained compiler/library paths from **Anaconda** and **MESA SDK**. These paths could override Homebrew or Apple compilers and confuse the Einstein Toolkit build system.

For the successful build, Conda was deactivated:

```bash
conda deactivate
```

MESA/MESA-SDK entries were also removed from the current shell environment during compilation so that the Einstein Toolkit could use the intended Homebrew compiler and libraries.

This does **not** uninstall MESA. It only prevents the MESA SDK environment from interfering with the ET compilation in that terminal session.

---

## 5. Downloading the Einstein Toolkit

The workshop instructions used the ET_2026_05 release manifest. [2]

The source was obtained with:

```bash
cd ~

curl -kLO https://raw.githubusercontent.com/gridaphobe/CRL/ET_2026_05/GetComponents

chmod a+x GetComponents

./GetComponents \
https://bitbucket.org/einsteintoolkit/manifest/raw/ET_2026_05/einsteintoolkit.th
```

This created the Cactus source tree:

```text
~/Cactus
```

The workshop explicitly asks participants to build the Einstein Toolkit before the numerical-relativity exercises and verify it with the supplied HelloWorld test. [2]

---

## 6. SimFactory setup

Inside the Cactus directory:

```bash
cd ~/Cactus

./simfactory/bin/sim setup-silent
```

**SimFactory** is a management layer used by the Einstein Toolkit to configure machines, build executables, create simulations, submit/run jobs, and manage simulation directories. [6]

A useful FLASH analogy is:

```text
SimFactory ≈ build/run manager around the Cactus executable
```

It plays a role somewhat similar to combining parts of FLASH's setup/build workflow with job/run management, although the exact architecture is different. [4][6]

---

## 7. Important macOS / Apple-Silicon fixes made during the build

The standard ET configuration did not compile cleanly on this M1 Mac with the current Homebrew/GCC stack, so several compatibility changes were made.

The main configuration file modified was:

```text
~/Cactus/configs/sim/OptionList
```

### 7.1 Use Homebrew CMake

The bundled/automatically detected CMake configuration caused trouble, so the Homebrew CMake installation was selected:

```text
CMAKE_DIR = /opt/homebrew/opt/cmake
```

This points the build system to the Homebrew CMake installation.

---

### 7.2 C++ compiler compatibility for NSIMD/macOS

The C++ flags were adjusted to expose required macOS declarations:

```text
CXXFLAGS = -g -std=c++17 -D_GNU_SOURCE -D_DARWIN_C_SOURCE
```

The important macOS-specific part is:

```text
-D_DARWIN_C_SOURCE
```

This was needed because an external dependency used by the ET build otherwise failed to see functions such as `quick_exit` / `at_quick_exit`. The Apple-Silicon ET build guide uses the same macOS compatibility idea. [5]

---

### 7.3 Temporarily disable OpenMP

A GCC internal compiler error occurred while compiling BHaHAHA code generated inside an OpenMP region.

For this workshop installation, OpenMP was therefore disabled:

```text
OPENMP = no
CPP_OPENMP_FLAGS =
FPP_OPENMP_FLAGS =
C_OPENMP_FLAGS =
CXX_OPENMP_FLAGS =
F90_OPENMP_FLAGS =
LD_OPENMP_FLAGS =
```

This is a **compatibility/workshop configuration**, not necessarily the best high-performance configuration.

For serious production calculations, especially on a cluster, OpenMP can be revisited later with a compiler/toolchain known to work reliably.

MPI remains available, so disabling OpenMP does not mean the executable is purely serial.

---

### 7.4 Open MPI C++ wrapper

The Open MPI C++ wrapper originally selected Apple `clang++`, while the ET build was using Homebrew GCC.

The working shell override was:

```bash
export OMPI_CXX=/opt/homebrew/bin/g++-15
```

This makes Open MPI's `mpicxx` wrapper use Homebrew `g++-15`.

A simple MPI program was compiled and executed successfully to verify that this configuration worked.

---

### 7.5 Explicit MPI location

The OptionList was given:

```text
MPI_DIR = /opt/homebrew/opt/open-mpi
```

This helps ET and external packages locate the intended Homebrew Open MPI installation.

---

### 7.6 Boost

The bundled Boost build failed with the modern compiler, so Homebrew Boost was used:

```text
BOOST_DIR = /opt/homebrew/opt/boost
BOOST_LIBS = boost_filesystem
```

This follows the same strategy recommended in the Apple-Silicon ET build guide. [5]

---

### 7.7 HDF5 / Silo / libaec (`libsz`)

Silo links against HDF5, and the Homebrew HDF5 package uses the `sz` compatibility library provided by `libaec`.

The required path was added:

```text
LIBSZ_DIR = /opt/homebrew/opt/libaec/lib
```

The library directory contains files such as:

```text
libsz.dylib
libsz.2.0.1.dylib
libaec.dylib
```

This solved the linker failure:

```text
ld: library 'sz' not found
```

The Apple-Silicon ET build guide explicitly recommends this `LIBSZ_DIR` configuration for Homebrew HDF5/libaec. [5]

---

### 7.8 Silo + modern GCC compatibility

Silo 4.11.1 produced errors such as:

```text
passing argument ... from incompatible pointer type
```

Modern GCC treats some old C constructs more strictly than older compilers.

The working C flags were:

```text
CFLAGS = -g -std=gnu99 \
-Wno-error=implicit-function-declaration \
-Wno-error=incompatible-pointer-types \
-DHDmemcmp=memcmp
```

These compatibility flags match the strategy used in the Apple-Silicon ET build configuration for Silo/HDF5 with recent GCC versions. [5]

---

## 8. Final successful build command

After the configuration fixes, the Einstein Toolkit was successfully built with:

```bash
cd ~/Cactus

export OMPI_CXX=/opt/homebrew/bin/g++-15

./simfactory/bin/sim build sim \
  --reconfig \
  --optionlist /Users/hope/Cactus/configs/sim/OptionList \
  --thornlist thornlists/einsteintoolkit.th \
  -j1
```

`-j1` means that the build used one parallel make job, which is slower but reduces memory usage and makes compiler failures easier to diagnose.

The workshop itself recommends reducing the number of build jobs if the build is killed or unstable. [2]

The final build ended with:

```text
Done.
```

and copied the required `hwloc` utilities into the simulation executable area.

---

## 9. HelloWorld test

The workshop-provided verification command is:

```bash
./simfactory/bin/sim create-run helloworld \
  --parfile arrangements/CactusExamples/HelloWorld/par/HelloWorld.par
```

The expected success message is:

```text
INFO (HelloWorld): Hello World!
```

The test ran successfully and printed that message repeatedly during the evolution loop. [2]

This confirms that:

```text
source code
   ↓
configuration
   ↓
compilation
   ↓
linking
   ↓
SimFactory
   ↓
parameter-file parsing
   ↓
Cactus scheduler
   ↓
runtime execution
```

all work correctly on this machine.

The workshop explicitly identifies the `INFO (HelloWorld): Hello World!` message as the success criterion. [2]

---

## 10. What exactly is `HelloWorld`?

`HelloWorld` is a very simple **Cactus thorn** included for demonstration/testing purposes. [3]

Its job is essentially to register a routine with the Cactus scheduler and print:

```text
Hello World!
```

during the evolution loop.

It is not a physics simulation.

Its purpose is analogous to compiling a minimal FLASH test problem to verify that the executable, runtime parameters, scheduler, and machine environment all work.

The corresponding parameter file is:

```text
~/Cactus/arrangements/CactusExamples/HelloWorld/par/HelloWorld.par
```

You can inspect it with:

```bash
cat ~/Cactus/arrangements/CactusExamples/HelloWorld/par/HelloWorld.par
```

---

## 11. What is a `.par` file?

A Cactus `.par` file is the runtime configuration for a simulation. [3]

This is the closest Einstein Toolkit analogue to a FLASH `flash.par`.

A parameter file may contain:

```text
ActiveThorns = "
    ...
"
```

which selects which already-compiled thorns are active during that run.

It also contains parameter assignments of the form:

```text
thorn_name::parameter_name = value
```

For example, conceptually:

```text
SomeThorn::resolution = ...
SomeThorn::final_time = ...
SomeIOThorn::output_every = ...
```

The exact parameters depend on which thorns are being used. [3]

---

## 12. What are `interface.ccl`, `param.ccl`, and `schedule.ccl`?

A typical Cactus thorn contains files such as:

```text
MyThorn/
├── interface.ccl
├── param.ccl
├── schedule.ccl
├── configuration.ccl
├── src/
└── par/
```

`interface.ccl` defines variables, groups, functions, and interfaces exposed by a thorn. [3]

`param.ccl` defines which runtime parameters the thorn provides. [3]

`schedule.ccl` tells Cactus **when** the thorn's routines should execute. [3]

`src/` contains the actual C/C++/Fortran implementation.

`par/` commonly contains example parameter files.

A useful FLASH analogy is:

```text
interface.ccl   → declares data/interface relationships
param.ccl       → defines runtime parameters
schedule.ccl    → decides when routines execute
src/            → numerical implementation
.par file       → chooses runtime configuration
```

---

## 13. Normal ET workflow

A normal Einstein Toolkit workflow looks like:

```text
1. Build an executable containing the required thorns
                  ↓
2. Choose or write a .par file
                  ↓
3. Activate/configure the required thorns
                  ↓
4. Create/run a simulation with SimFactory
                  ↓
5. Cactus initializes the grid and variables
                  ↓
6. Initial data are constructed/read
                  ↓
7. Evolution equations are advanced in time
                  ↓
8. Diagnostics and output routines are executed
                  ↓
9. Checkpoints/output files are written
                  ↓
10. Analyse and visualize the results
```

This is conceptually very similar to the workflow of setting up a FLASH problem, compiling it, choosing `flash.par`, running `flash4`, and then analysing checkpoint/plot files. [4]

---

## 14. Starting ET in a new terminal

The ET executable is already built, so **you do not rebuild it every time**.

A safe startup sequence is:

```bash
conda deactivate 2>/dev/null

export PATH="/usr/sbin:/opt/homebrew/bin:$PATH"

export OMPI_CXX=/opt/homebrew/bin/g++-15

cd ~/Cactus
```

For simply running an existing executable, the `OMPI_CXX` setting is usually not important, but keeping it set is convenient if you later rebuild or compile something.

---

## 15. Running HelloWorld again

SimFactory stores a simulation under a simulation name.

Because:

```text
helloworld
```

already exists, trying to create it again gives:

```text
Simulation ".../simulations/helloworld" already exists
```

Use a new name:

```bash
./simfactory/bin/sim create-run helloworld2 \
  --parfile arrangements/CactusExamples/HelloWorld/par/HelloWorld.par
```

or:

```bash
./simfactory/bin/sim create-run test1 \
  --parfile arrangements/CactusExamples/HelloWorld/par/HelloWorld.par
```

The name `helloworld2`, `test1`, etc. is only the **simulation name**; the physics/setup comes from the `.par` file.

---

## 16. Where simulations are stored

For the local SimFactory setup used here, simulations are stored under:

```text
~/simulations/
```

For example:

```text
~/simulations/helloworld/
~/simulations/helloworld2/
```

SimFactory manages these directories and keeps run/output information associated with each simulation. [6]

---

## 17. What can actually be done with the Einstein Toolkit?

The Einstein Toolkit is primarily designed for **general-relativistic simulations**. [1]

Typical applications include:

- binary black-hole mergers,
- binary neutron-star mergers,
- black-hole–neutron-star systems,
- relativistic hydrodynamics,
- general-relativistic magnetohydrodynamics,
- gravitational collapse,
- compact-star oscillations,
- rotating relativistic stars,
- apparent-horizon finding,
- gravitational-wave extraction,
- numerical tests of Einstein's equations,
- relativistic astrophysical fluid flows,
- mesh-refined simulations of compact objects. [1]

The exact capabilities depend on which thorns are included and maintained in the chosen ET release. [1]

---

## 18. Why ET is useful if you already know FLASH

FLASH is extremely useful for Newtonian and non-relativistic astrophysical fluid dynamics/MHD and contains extensive hydrodynamics, gravity, EOS, AMR, particle, nuclear-burning, and multiphysics infrastructure. [4]

The Einstein Toolkit becomes particularly useful when the **spacetime itself must evolve dynamically according to Einstein's equations**. [1]

The key conceptual difference is:

```text
FLASH:
matter evolves mainly on a prescribed/Newtonian gravitational background

Einstein Toolkit:
matter + spacetime can both be evolved dynamically
```

This is a simplified comparison, because FLASH has many gravity capabilities and ET can also run fixed-background problems, but it captures the main reason numerical relativists use the Einstein Toolkit. [1][4]

In general relativity, quantities such as the metric are dynamical fields.

A typical ET simulation may therefore evolve:

```text
spacetime metric
       +
curvature variables
       +
density
       +
pressure
       +
velocity
       +
magnetic field
```

depending on the formulation and physics modules being used. [1]

For someone coming from FLASH MHD, the additional major concept is therefore that **gravity is no longer just a source term or Poisson solve; the geometry of spacetime itself becomes part of the PDE system**. [1]

---

## 19. FLASH vs Einstein Toolkit: simulation structure

A rough conceptual comparison is:

```text
FLASH MHD example
-----------------
ρ
P
v
B
Φ / gravity
       ↓
MHD solver
       ↓
AMR
       ↓
checkpoint / plot files
```

versus:

```text
Einstein Toolkit GR/GRMHD example
---------------------------------
ρ, P, v, B
        +
spacetime metric
        +
extrinsic curvature / formulation variables
        ↓
GR + GRMHD evolution
        ↓
mesh refinement
        ↓
horizon / wave / fluid diagnostics
        ↓
checkpoint / output
```

The actual variables and evolution system depend on the formulation and thorns used. [1]

---

## 20. Example of how a real ET simulation differs from HelloWorld

The HelloWorld simulation has essentially:

```text
Active thorn:
    HelloWorld

Evolution action:
    print a message
```

A real numerical-relativity simulation might conceptually activate thorns responsible for:

```text
spacetime evolution
hydrodynamics
initial data
mesh refinement
boundary conditions
I/O
apparent horizons
gravitational-wave extraction
```

Those thorns communicate through Cactus variables and are executed according to the scheduler. [3]

---

## 21. Build-time selection vs runtime selection

This distinction is important.

### Build time

The thorn list:

```text
thornlists/einsteintoolkit.th
```

determines which thorns are compiled into the executable.

### Runtime

The `.par` file determines which of those compiled thorns are actually activated and what parameter values they use. [3]

The analogy with FLASH is approximately:

```text
FLASH Config / setup units
        ↕
ET thorn list

FLASH flash.par
        ↕
ET .par file
```

---

## 22. The build command vs the run command

### Build

```bash
./simfactory/bin/sim build sim \
  --reconfig \
  --optionlist ~/Cactus/configs/sim/OptionList \
  --thornlist thornlists/einsteintoolkit.th \
  -j1
```

This creates/updates the Cactus executable.

### Run

```bash
./simfactory/bin/sim create-run NAME \
  --parfile PATH/TO/file.par
```

This creates and executes a simulation using an already-built executable. [6]

You usually build far less often than you run.

That is similar to FLASH: compile `flash4` once for a problem configuration, then run multiple cases with different runtime parameters.

---

## 23. Current known-good local configuration

The following settings were important for this Mac:

```text
CMAKE_DIR = /opt/homebrew/opt/cmake

CXXFLAGS = -g -std=c++17 -D_GNU_SOURCE -D_DARWIN_C_SOURCE

OPENMP = no
CPP_OPENMP_FLAGS =
FPP_OPENMP_FLAGS =
C_OPENMP_FLAGS =
CXX_OPENMP_FLAGS =
F90_OPENMP_FLAGS =
LD_OPENMP_FLAGS =

MPI_DIR = /opt/homebrew/opt/open-mpi

BOOST_DIR = /opt/homebrew/opt/boost
BOOST_LIBS = boost_filesystem

LIBSZ_DIR = /opt/homebrew/opt/libaec/lib

CFLAGS = -g -std=gnu99 \
-Wno-error=implicit-function-declaration \
-Wno-error=incompatible-pointer-types \
-DHDmemcmp=memcmp
```

The shell used:

```bash
export OMPI_CXX=/opt/homebrew/bin/g++-15
```

These settings produced a successful ET_2026_05 build and a successful `HelloWorld` run on this M1 Mac.

---

## 24. Important caveat for future work

The current build has **OpenMP disabled** because of a compiler-specific internal error encountered with the BHaHAHA code.

That is acceptable for learning, the workshop, and local testing.

For large production simulations, especially on HPC systems, use the machine's recommended compiler/MPI stack and create a machine-specific ET configuration rather than copying this Mac configuration directly.

Your IIT Bombay cluster configuration, for example, should be treated separately from this laptop configuration.

---

## 25. What not to do in a fresh terminal

Do not rerun `GetComponents` every time.

Do not rebuild the full Toolkit every time.

Do not reinstall Homebrew packages every time.

Do not recreate the same SimFactory simulation name unless you intentionally remove/purge the old simulation first.

Normally you only need to enter the Cactus directory and run a new/existing parameter file.

---

## 26. Minimal daily-use cheat sheet

### Start a clean terminal environment

```bash
conda deactivate 2>/dev/null
export PATH="/usr/sbin:/opt/homebrew/bin:$PATH"
export OMPI_CXX=/opt/homebrew/bin/g++-15
cd ~/Cactus
```

### Inspect a parameter file

```bash
cat arrangements/CactusExamples/HelloWorld/par/HelloWorld.par
```

### Run a new simulation

```bash
./simfactory/bin/sim create-run myrun \
  --parfile arrangements/CactusExamples/HelloWorld/par/HelloWorld.par
```

### Expected HelloWorld success message

```text
INFO (HelloWorld): Hello World!
```

### Rebuild only when necessary

```bash
./simfactory/bin/sim build sim \
  --reconfig \
  --optionlist /Users/hope/Cactus/configs/sim/OptionList \
  --thornlist thornlists/einsteintoolkit.th \
  -j1
```

---

## 27. The one-picture mental model

```text
                 EINSTEIN TOOLKIT
                        |
                        v
                Cactus Framework
                        |
          +-------------+-------------+
          |             |             |
        Thorn         Thorn         Thorn
       Physics        AMR/I/O      Diagnostics
          \             |             /
           \            |            /
            +-----------+-----------+
                        |
                  parameter file
                     (.par)
                        |
                        v
                    SimFactory
                        |
                        v
                 Cactus executable
                        |
                        v
                   simulation
                        |
             +----------+----------+
             |                     |
          output               checkpoints
             |
             v
        analysis/plots
```

For a FLASH user, the closest compact analogy is:

```text
Cactus        ≈ FLASH framework
thorn         ≈ FLASH Unit/module
.par file     ≈ flash.par
SimFactory    ≈ setup/run/job manager
ET executable ≈ flash4
Carpet(X)     ≈ AMR infrastructure
ET output     ≈ plot/checkpoint data
```

Again, these are conceptual analogies rather than one-to-one software equivalences. [1][3][4][6]

---

## 28. What this setup now enables for the workshop

The laptop now satisfies the workshop's core Einstein Toolkit preparation requirement: the Toolkit has been compiled successfully and the `HelloWorld` parameter file has been run successfully. [2]

This gives a working local environment in which the numerical-relativity exercises can be run, parameter files can be modified, Cactus scheduling can be explored, and later relativistic simulations can be analysed. [1][2]

The next useful learning step is not more installation work: it is to inspect `HelloWorld.par`, then inspect the corresponding thorn's `param.ccl`, `schedule.ccl`, and source file so that the relationship between a parameter file, a thorn, and the Cactus scheduler becomes concrete. [3]

---

# References

[1] **Einstein Toolkit official website / documentation**  
https://einsteintoolkit.org/

[2] **GR Multimessenger workshop/course preparation document supplied for the course**  
The supplied course instructions require the Einstein Toolkit to be built before arrival and use the HelloWorld example as the installation check.

[3] **Cactus Users Guide**  
https://einsteintoolkit.org/usersguide/UsersGuide.html

[4] **FLASH Center for Computational Science — FLASH Code**  
https://flash.rochester.edu/site/flashcode/

[5] **Apple-Silicon Einstein Toolkit compile guide / macOS configuration**  
https://github.com/lwJi/ETK-Compile-Guides/tree/main/macos

[6] **SimFactory documentation**  
https://simfactory.bitbucket.io/
