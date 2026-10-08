# Einstein Toolkit: Short Static TOV Test on macOS (Apple Silicon)

## Goal

The aim of this test was to go beyond the `HelloWorld` example and run a small, real numerical-relativity problem with the Einstein Toolkit.

We used the standard static TOV-star example, but shortened the evolution so that it could be tested quickly on the MacBook.

---

## 1. Starting parameter file

The original Einstein Toolkit parameter file was:

```bash
~/Cactus/par/static_tov.par
```

A short-test copy was made so that the original file stayed unchanged:

```bash
cp ~/Cactus/par/static_tov.par ~/Cactus/par/static_tov_short.par
```

For the short test, the final evolution time was changed to:

```text
Cactus::cctk_final_time = 10
```

The scalar-output frequency was also increased so that enough diagnostic points were written during the short run:

```text
IOScalar::outScalar_every = 8
```

The rest of the physical TOV setup was kept unchanged.

---

## 2. Initial problem on the Mac

The first short TOV runs crashed during initialization with:

```text
Assertion failed: (imin < imax), function randomui,
file loopcontrol.cc, line 325.
```

The backtrace showed that the failure occurred inside `LoopControl`, through:

```text
randomui(...)
  -> LC_control_init
  -> MaskBase_InitMask
```

This happened before the actual TOV evolution started.

Trying one MPI process and using only:

```text
LoopControl::initial_setup = "legacy"
LoopControl::settle_after_iteration = 0
```

was not enough; the same assertion still occurred.

One later test also stopped because some LoopControl parameters had accidentally been written twice in the parameter file. That was a parameter-file error, not a physics or evolution error.

---

## 3. Working parameter-file changes

A fresh parameter file was made from the clean short version:

```bash
cp ~/Cactus/par/static_tov_short.par \
   ~/Cactus/par/static_tov_short_mac2.par
```

The following settings were added once at the end:

```text
# Apple-Silicon LoopControl workaround
LoopControl::initial_setup = "legacy"
LoopControl::settle_after_iteration = 0
LoopControl::random_jump_probability = 0.0
LoopControl::align_with_cachelines = no
```

These settings provided a working runtime workaround for the LoopControl initialization problem on this machine.

No source-code patch was needed.

---

## 4. Successful run command

The successful short TOV test was launched from `~/Cactus` with:

```bash
./simfactory/bin/sim create-run static_tov_short_mac2 \
    --parfile=par/static_tov_short_mac2.par \
    --procs=1 \
    --num-threads=1 \
    --ppn-used=1
```

This used:

- 1 MPI process
- 1 thread
- the short final time `t = 10`

---

## 5. What was active in the successful run

Important thorns activated during the run included:

```text
TOVSolver
GRHydro
HydroBase
EOS_Polytrope
ADMBase
ML_BSSN
ML_ADMConstraints
MoL
Carpet
CarpetRegrid2
CarpetIOScalar
CarpetIOASCII
CarpetIOHDF5
LoopControl
```

In simple terms:

- `TOVSolver` constructed the initial neutron-star model.
- `GRHydro` evolved the relativistic fluid.
- `ML_BSSN` evolved the spacetime variables.
- `MoL` supplied the time-integration framework.
- `Carpet` handled the mesh-refinement hierarchy and data movement.
- the Carpet I/O thorns wrote scalar and spatial output.

---

## 6. Grid used in the test

The run created five Carpet refinement levels.

The reported grid spacings were approximately:

```text
Level 0 : dx = 8
Level 1 : dx = 4
Level 2 : dx = 2
Level 3 : dx = 1
Level 4 : dx = 0.5
```

So the finest grid spacing was:

```text
dx = 0.5
```

The successful one-process run reported about:

```text
0.91 GB
```

of required memory for the active grid functions and arrays.

---

## 7. Successful termination

The run reached:

```text
Iteration = 2560
Time      = 10.000
```

The last printed diagnostics were approximately:

```text
ADMBase::alp
minimum = 0.6698054
maximum = 0.9966375

HydroBase::rho
minimum = 1.000000e-10
maximum = 0.0012781
```

The run then ended normally with:

```text
INFO (Carpet): Terminating due to cctk_final_time at t = 10.000000
```

This means the short test finished because it reached the requested final time, not because of a crash.

---

## 8. Output directory

The successful run output is stored in:

```bash
~/simulations/static_tov_short_mac2/output-0000/static_tov_short_mac2/
```

Important scalar files include:

```text
hydrobase-rho.maximum.asc
hydrobase-rho.minimum.asc

admbase-lapse.maximum.asc
admbase-lapse.minimum.asc

ml_admconstraints-ml_ham.maximum.asc
ml_admconstraints-ml_ham.norm2.asc
```

Important 1-D spatial files include:

```text
hydrobase-rho.x.asc
hydrobase-rho.y.asc
hydrobase-rho.z.asc
```

These files can be used next to study the density evolution, lapse evolution, constraints, and density profile.

---

## 9. Failed runs that can be removed

The failed test simulations were:

```text
static_tov_short
static_tov_short_legacy
static_tov_short_mac
```

The successful simulation that should be kept is:

```text
static_tov_short_mac2
```

The clean short parameter file and successful parameter file should also be kept:

```text
~/Cactus/par/static_tov_short.par
~/Cactus/par/static_tov_short_mac2.par
```

The failed intermediate parameter files can be removed after confirming they are no longer needed:

```text
~/Cactus/par/static_tov_short_legacy.par
~/Cactus/par/static_tov_short_mac.par
```

---

## 10. Next analysis step

The first useful diagnostic will be the relative change of maximum density,

\[
\frac{\rho_{\max}(t)-\rho_{\max}(0)}
     {\rho_{\max}(0)},
\]

followed by:

1. minimum lapse versus time,
2. Hamiltonian-constraint diagnostics,
3. the 1-D density profile.

For now, the important result is that the Einstein Toolkit installation can successfully initialize and evolve the static TOV problem on this Mac using the working LoopControl settings above.
