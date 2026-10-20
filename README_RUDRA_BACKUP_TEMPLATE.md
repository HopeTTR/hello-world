# Param Rudra — simulation data inventory and backup request

**Prepared:** 2026-10-16  
**Research area:** Computational astrophysics — magnetised white-dwarf structure and dynamical MHD simulations  
**Software used in this project:** FLASH 4.7.1; MESA 24.08.1 (verify exact version for each run)  
**Supervisor:** Prof. Rahul Kashyap  
**Rudra account:** `23n0315`  
**Scratch project root (previously used; confirm current location):** `/scratch/IITB/CompnalAstrphyNRelvity/23n0315/`  
**Additional project / MESA files:** `/home/IITB/CompnalAstrphyNRelvity/23n0315/` (check specific directories)

## 1. Overview

My simulations involve constructing white-dwarf initial models using MESA, mapping the stellar structure onto a three-dimensional FLASH grid, checking hydrodynamical relaxation, and investigating rotation and magnetic-field configurations. This inventory identifies the results and exact inputs that should be backed up before the Param Rudra maintenance. It is **not** a claim that any specific run has already been archived.

## 2. Inventory of simulation runs (fill using current cluster output)

| Run ID | Absolute Rudra path | Size | Physical purpose / key parameters | Status (complete/running/failed) | Importance (1–4) | Backup request | Reproducible? | Notes |
|---|---|---|---|---|---|---|---|---|
| `run_17` (previously analysed; verify current location) | `<absolute path>` | `<GB>` | `<e.g., hydro / rotation / MHD settings; verify>` | `<status>` | `<priority>` | `<yes/no>` | `<yes/no; prerequisites>` | `<plots/checkpoints/results>` |
| `<run ID>` | `<absolute path>` | `<GB>` | `<purpose, B0, r0, rotation profile, damping, resolution>` | `<status>` | `<priority>` | `<yes/no>` | `<yes/no>` | `<important products>` |
| `<run ID>` | `<absolute path>` | `<GB>` | `<purpose, parameters>` | `<status>` | `<priority>` | `<yes/no>` | `<yes/no>` | `<important products>` |

Priority definitions (from group guidelines): **1** irreplaceable/paper; **2** active analysis/keep; **3** regenerable with complete provenance and reduced products; **4** scratch/test/failed or superseded.

If total data exceed available backup space, prioritise the essential files and selected scientifically important outputs below. Do **not** delete original data before a verified copy exists.

## 3. File types, meaning, and what should be preserved

| File / directory type | Scientific purpose | Backup priority |
|---|---|---|
| FLASH runtime parameter file (often `flash.par`) and build/setup configuration | Exact physical and numerical settings | Essential |
| FLASH custom source code (`Simulation_initBlock.F90`, `Simulation_adjustEvolution.F90`, related units) and Git commit/diff | Reproduces custom initial conditions, damping, magnetic/rotation setup | Essential |
| SLURM job scripts, standard-output/error logs, compiler and MPI module versions | Execution environment, job allocation and failures | Essential |
| MESA input files (`inlist*`), profiles and source/model generation script | Recreates the 1D white dwarf used for mapping | Essential |
| `*_hdf5_chk_*` | FLASH checkpoints for restarting a run | Important for selected runs |
| `*_hdf5_plt_cnt_*` | FLASH field snapshots for analysis | Selected outputs or complete series for high-value runs |
| `*_hdf5_part_*` | Particle output (if present and scientifically used) | Depends on run |
| `reduced/` (time series, radial profiles, spectra, representative slices) and plot scripts | Results used in presentations, papers and analysis | Essential; small enough to retain broadly |
| `PROVENANCE.txt`, `MANIFEST.sha256`, `MANIFEST.info` | Exact inputs/commits and checksum integrity | Essential |

Keep raw output read-only after the run finishes; write plots and other derived products to `reduced/` or separate analysis directories.

## 4. Reproducibility details — complete for each run

For every run, preserve or explicitly record:

- **Exact FLASH code**: repository URL, commit hash, branch, and any uncommitted diff; source changes to initialisation/MHD units.
- **Build**: FLASH setup command, site, compiler, MPI, modules (`module list`), flags, executable identity.
- **Run settings**: unchanged parameter file as used, box dimensions, AMR/max refinement, EOS, gravity, CFL, rotation profile, magnetic parameters (`B0`, `r0`) and damping when applicable.
- **Initial conditions**: MESA version, input `inlist*`, exact `profile*.data`, mapping program/script and generation instructions.
- **Execution**: SLURM script, nodes/tasks, runtime, run command, output/checkpoint cadence and status.
- **Analysis**: scripts and input files producing each published/report figure; saved reduced data and their code version.
- **Restart or reproduction command**: exact compile/submit/restart steps plus prerequisites and expected output. Record when details are unavailable.

Use the provided `compastro-DataManifest.sh` script's `provenance`, `create` and `verify` commands after filling in source-code and input paths. The manifest should be created *after* provenance and reduced products are added.

## 5. Key products requested for backup

1. **Reproducibility bundle for all scientifically meaningful runs**: parameter files, job scripts, source modifications/commits, MESA initial model, mapping scripts, logs and analysis code.
2. **High-value results for active analysis**: selected checkpoints, relevant FLASH plot files, and reduced diagnostics (e.g., central density variation, kinetic/magnetic energies, radial density profiles).
3. **Full raw runs only where justified**: identify them explicitly by run ID, size, priority and rationale in the table above.

**Total size requested for backup:** `<compute>`  
**Second-storage destination / status:** `<NAS project folder, private NAS folder, department-cluster path, or TBD>`  
**Copy verified:** `<yes/no; date and method>`  
**Known missing ingredients / limitations:** `<anything that prevents exact reproduction>`

## 6. Verification and follow-up

- Calculate and record SHA-256 manifests for finished datasets; verify them after copying.
- Update `RUNS.tsv` with directory, size, priority, code commit and status; store it with project code, not only on scratch.
- Ask the supervisor/administrator to confirm which high-priority runs can be stored on the group NAS or another storage system.
- Do not delete source copies until a verified second copy exists.

**Reference documents:** `CompAstroGroup-Data-Guidelines.md` and `CompAstroGroup-Student-Guide.md` (both updated 2026-10-14), plus the provided `compastro-DataManifest.sh` script.
