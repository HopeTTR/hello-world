# Wolfram Engine + VS Code Setup Summary (macOS Apple Silicon)

> **Machine:** MacBook Air, Apple Silicon (ARM64)  
> **Wolfram Engine:** 15.0.0  
> **VS Code:** 1.140.0, ARM64  
> **Purpose:** Prepare the Wolfram Language environment required for the Black Hole Perturbation Theory part of the GR Multimessenger course. The course explicitly allows **Wolfram Engine 12+ with VS Code and the Wolfram Language extension** as an alternative to Mathematica. [1]

---

## 1. Why this setup was needed

The Black Hole Perturbation Theory part of the course expects access to **Wolfram Mathematica 10+**, or alternatively **Wolfram Engine 12+** together with **VS Code + the Wolfram Language extension**, or Jupyter with a Wolfram Language kernel. [1]

Because Mathematica was not installed on this Mac, the free **Wolfram Engine for Developers** route was used. [1][2]

---

## 2. Check that Mathematica / Wolfram was not already installed

The first checks were:

```bash
ls /Applications | grep -i Mathematica
which wolframscript
```

Both returned nothing, confirming that Mathematica and `wolframscript` were not installed.

---

## 3. Install Wolfram Engine with Homebrew

The Wolfram Engine was installed with:

```bash
brew install --cask wolfram-engine
```

The installer downloaded about 3.1 GB and installed:

```text
/Applications/Wolfram Engine.app
```

Homebrew also linked:

```text
wolframscript
```

into the command-line environment.

The successful installation message was:

```text
wolfram-engine was successfully installed!
```

The free Wolfram Engine developer license is obtained through Wolfram's free-license page. [2]

---

## 4. Disk-space issue encountered during installation

One installation attempt failed with:

```text
No space left on device
```

This happened while Homebrew was unpacking the Wolfram Engine application.

After freeing additional disk space, the same install command was rerun:

```bash
brew install --cask wolfram-engine
```

and the installation completed successfully.

For a large macOS cask like Wolfram Engine, the machine needs enough free space for both the downloaded installer and temporary unpacking.

---

## 5. `wolframscript` initially could not locate the kernel

After installation:

```bash
wolframscript
```

returned:

```text
A WolframKernel location could not be determined.
Use -configure to set WOLFRAMSCRIPT_KERNELPATH.
```

The installed kernel path was configured manually:

```bash
wolframscript -config \
WOLFRAMSCRIPT_KERNELPATH="/Applications/Wolfram Engine.app/Contents/MacOS/WolframKernel"
```

This produced:

```text
Configured:WOLFRAMSCRIPT_KERNELPATH=/Applications/Wolfram Engine.app/Contents/MacOS/WolframKernel
```

---

## 6. Activate the free Wolfram Engine license

Running:

```bash
wolframscript
```

then requested one-time activation.

The Wolfram ID was associated with the free developer license through Wolfram's license page. [2]

Activation was completed with:

```bash
wolframscript -activate
```

The successful result was:

```text
Wolfram Engine activated.
```

The non-interactive test:

```bash
wolframscript -code '2+2'
```

returned:

```text
4
```

This confirmed that the Engine was activated and usable from the command line.

---

## 7. Check the license directly

The license information was checked using:

```bash
"/Applications/Wolfram Engine.app/Contents/MacOS/WolframKernel" -licenseinfo
```

which returned a valid license entry.

This confirmed that the Wolfram kernel had access to a valid license.

---

## 8. Test the Wolfram Language interactively

Running:

```bash
wolframscript
```

opened an interactive Wolfram Language session:

```text
Wolfram Language 15.0.0 Engine for Mac OS X ARM (64-bit)

In[1]:=
```

A simple calculation was tested:

```wolfram
2 + 2
```

and returned:

```text
Out[1]= 4
```

The session was exited with:

```wolfram
Quit[]
```

---

## 9. VS Code was already installed

VS Code was present at:

```text
/Applications/Visual Studio Code.app
```

but initially the shell command:

```bash
code
```

was not available.

The `code` CLI path was added permanently to `~/.zprofile`:

```bash
echo 'export PATH="$PATH:/Applications/Visual Studio Code.app/Contents/Resources/app/bin"' >> ~/.zprofile
```

Then:

```bash
source ~/.zprofile
```

After that:

```bash
which code
```

returned:

```text
/Applications/Visual Studio Code.app/Contents/Resources/app/bin/code
```

and:

```bash
code --version
```

returned VS Code version `1.140.0` on `arm64`.

---

## 10. Install the official Wolfram Language VS Code extension

The official Wolfram Research extension was installed with:

```bash
code --install-extension WolframResearch.wolfram
```

The successful installation message was:

```text
Extension 'wolframresearch.wolfram' v3.0.2 was successfully installed.
```

Verification:

```bash
code --list-extensions | grep -i wolfram
```

returned:

```text
wolframresearch.wolfram
```

The extension is the official Wolfram Language extension for VS Code. [3]

---

## 11. Configure the Wolfram system path in VS Code

Inside VS Code:

```text
Settings
→ search: Wolfram: System Path
```

The final working setting was:

```text
/Applications/Wolfram Engine.app
```

rather than leaving it on:

```text
Automatic
```

The current extension source recognizes Wolfram Engine installations on macOS and resolves the actual nested kernel path when an Engine app path is supplied. [3]

For the current Homebrew installation, the extension resolves the Engine to a path inside:

```text
/Applications/Wolfram Engine.app/
Contents/Resources/Wolfram Player.app/Contents/MacOS/WolframKernel
```

The extension's kernel-finding code contains explicit macOS handling for Wolfram Engine installations. [3]

---

## 12. VS Code kernel initially failed because the Engine was not fully activated

Before the final activation fix, VS Code reported:

```text
The terminal process "...WolframKernel" failed to launch (exit code: 1).
```

At the same time:

```bash
wolframscript -code '2+2'
```

reported:

```text
Your Wolfram Engine installation is not activated
or is experiencing a license-related problem.
```

This showed that the problem was not primarily the VS Code path; the Engine license had to be activated again.

The fix was:

```bash
wolframscript -activate
```

After activation:

```bash
wolframscript -code '2+2'
```

returned:

```text
4
```

and VS Code could then launch the Wolfram kernel successfully.

---

## 13. Working VS Code test

A small Wolfram Language file was created in the workshop workspace:

```text
test.wl
```

The workspace used in this setup is under the user's software directory:

```text
~/sw/wolfram_workshop/
```

The file contained:

```wolfram
2 + 2
```

The line was selected in VS Code and evaluated with:

```text
Shift + Enter
```

The Wolfram terminal opened inside VS Code and showed:

```text
In[1]:= 2+2

Out[1]= 4
```

This confirmed that the complete chain is working:

```text
VS Code
   ↓
Wolfram Language extension
   ↓
Wolfram Engine kernel
   ↓
valid developer license
   ↓
Wolfram Language evaluation
```

---

## 14. Current working setup

The main components now working are:

```text
Wolfram Engine 15.0.0
        +
Free Wolfram Engine developer license
        +
wolframscript
        +
VS Code 1.140.0 ARM64
        +
WolframResearch.wolfram extension
```

The course requirement for the Wolfram/Mathematica software environment is therefore satisfied. [1]

---

## 15. Daily-use workflow

For command-line Wolfram Language work:

```bash
wolframscript
```

For one-line calculations:

```bash
wolframscript -code '2+2'
```

For a script file:

```bash
wolframscript -file myscript.wl
```

For VS Code work:

```bash
cd ~/sw/wolfram_workshop
code .
```

Then open a `.wl` file, select a line or expression, and press:

```text
Shift + Enter
```

The official Wolfram extension provides a command to run selected text in a dedicated Wolfram terminal. [3]

---

## 16. Useful first tests

Arithmetic:

```wolfram
2 + 2
```

Symbolic differentiation:

```wolfram
D[Sin[x]^2, x]
```

Symbolic integration:

```wolfram
Integrate[Exp[-x^2], {x, -Infinity, Infinity}]
```

Expected results are equivalent to:

```text
4
2 Cos[x] Sin[x]
Sqrt[Pi]
```

These are useful sanity checks for symbolic algebra, which is one of the main reasons Wolfram Language is useful in perturbation-theory calculations.

---

## 17. Why Wolfram Language is useful for the course

For Black Hole Perturbation Theory, symbolic algebra is useful for operations such as:

```text
differentiation
series expansion
solving algebraic equations
tensor/component manipulation
simplifying expressions
ODE manipulation
special functions
asymptotic expansions
```

The course specifically asks participants to have basic familiarity with Mathematica/Wolfram Language syntax and grammar before the tutorials. [1]

---

## 18. Relation to the Einstein Toolkit setup

The Wolfram setup and Einstein Toolkit setup serve different purposes.

```text
Einstein Toolkit
→ numerical-relativity simulations
→ evolve PDEs on a computational grid

Wolfram Engine
→ symbolic / analytic calculations
→ manipulate equations and expressions
```

For a FLASH user, a useful analogy is:

```text
FLASH / Einstein Toolkit
→ simulation code

Wolfram Language
→ symbolic calculation / derivation / notebook tool
```

So Wolfram Engine is not replacing FLASH or the Einstein Toolkit; it complements them by helping with mathematical derivations and analytic manipulations.

---

## 19. Important commands to remember

### Activate again if licensing ever fails

```bash
wolframscript -activate
```

### Check basic functionality

```bash
wolframscript -code '2+2'
```

### Open an interactive session

```bash
wolframscript
```

### Check license information

```bash
"/Applications/Wolfram Engine.app/Contents/MacOS/WolframKernel" -licenseinfo
```

### Open the workshop folder in VS Code

```bash
cd ~/sw/wolfram_workshop
code .
```

### Check extension installation

```bash
code --list-extensions | grep -i wolfram
```

---

## 20. Final status

The final verified state is:

```text
Wolfram Engine installed      ✓
Developer license activated   ✓
wolframscript working         ✓
Command-line calculation      ✓
VS Code installed             ✓
`code` command working        ✓
Wolfram extension installed   ✓
Wolfram system path set       ✓
VS Code kernel launches       ✓
2+2 evaluated in VS Code      ✓
```

The Wolfram prerequisite is therefore complete.

---

# References

[1] **GR Multimessenger course information supplied for the workshop**  
The course software-requirements section states that participants should have Mathematica 10+ or may use Wolfram Engine 12+ with VS Code and the Wolfram Language extension.

[2] **Wolfram Engine for Developers / activation support**  
https://www.wolfram.com/engine/  
https://www.wolfram.com/engine/free-license/  
https://support.wolfram.com/46070

[3] **Official Wolfram Language extension for Visual Studio Code**  
https://github.com/WolframResearch/vscode-wolfram  
https://marketplace.visualstudio.com/items?itemName=WolframResearch.wolfram

[4] **Visual Studio Code macOS command-line setup**  
https://code.visualstudio.com/docs/setup/mac
