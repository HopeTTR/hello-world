# Param Rudra — FLASH white-dwarf simulation data README

**Prepared for:** Prof. Rahul Kashyap / CompAstroGroup, IIT Bombay  
**Research area:** Computational astrophysics; numerical evolution of white dwarfs using FLASH (hydrodynamics, rotation and magnetohydrodynamics). MESA stellar models are used for initial conditions.  
**Research user:** `23n0315`  
**Original inventory:** 2026-10-16 (43 directories)  
**Updated inventory:** Screenshot supplied 2026-10-21 (30 remaining listed run directories; inventory command date unverified)  
**README updated:** 2026-10-21  
**Scratch root:** `/scratch/IITB/CompnalAstrphyNRelvity/23n0315`  
**Non-scratch home directory:** `/home/IITB/CompnalAstrphyNRelvity/23n0315/`  
**Backup status:** Planned only; not confirmed transferred or verified.

## 1. Summary and storage request

- Request transfer of **the five largest currently listed directories**: `run_17`, `run_47`, `run_43`, `run_16`, `run_19`.
- The five now occupy **555.828 GiB (0.555 TiB)**, accounting for **55.0%** of the listed total. The other **27** directories contain **463.709 GiB** and are **not part of the present transfer request**.
- Transfers aim to protect datasets and, **only after verified backup and explicit confirmation**, permit freeing scratch storage for new simulations. This README neither authorises nor records any deletion.
- Selection is by occupied space, **not scientific retention priority**. The scientific value and reproducibility of each run should be assessed separately.


## 2. Five current largest directories proposed for backup

| Run | Size (GiB) | Total files | FLASH plot files | FLASH checkpoints | Current action |
|---|---:|---:|---:|---:|---|
| `run_17` | 161.030 | 372 | 338 | 9 | Transfer planned; not verified |
| `run_47` | 152.469 | 953 | 907 | 7 | Transfer planned; not verified |
| `run_43` | 110.386 | 360 | 319 | 8 | Transfer planned; not verified |
| `run_16` | 62.192 | 723 | 690 | 2 | Transfer planned; not verified |
| `run_19` | 69.751 | 169 | 151 | 3 | Transfer planned; not verified |
| **TOTAL** | **555.828** | **2,577** | **2405** | **29** | **Not yet transferred** |

Counts are inferred from filenames by the inventory script; they are not an integrity verification. All directories are located at `/scratch/IITB/CompnalAstrphyNRelvity/23n0315/<run_id>`.

## 3. All 30 run directories in the updated inventory 

The table preserves the current inventory of all identified simulation folders, including the 38 **not selected** for transfer.

| Run | Size (GiB) | Files | Plot files | Checkpoints | Particle files | Param. files | Job scripts | Log files | Transfer |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---|
| `run02` | 12.840 | 130 | 19 | 3 | 93 | 1 | 1 | 1 | No |
| `run03` | 1.200 | 27 | 2 | 1 | 2 | 1 | 1 | 5 | No |
| `run_10` | 8.561 | 249 | 206 | 3 | 0 | 3 | 3 | 3 | No |
| `run_11` | 5.494 | 104 | 72 | 2 | 0 | 2 | 2 | 2 | No |
| `run_12` | 11.882 | 46 | 25 | 3 | 0 | 1 | 1 | 1 | No |
| `run_13` | 12.227 | 45 | 26 | 3 | 2 | 1 | 1 | 1 | No |
| `run_14` | 14.894 | 401 | 369 | 2 | 0 | 2 | 2 | 2 | No |
| `run_15` | 5.478 | 24 | 8 | 2 | 0 | 1 | 1 | 1 | No |
| `run_16` | 62.192 | 723 | 690 | 2 | 0 | 2 | 2 | 2 | **YES — top five** |
| `run_17` | 161.030 | 372 | 338 | 9 | 0 | 1 | 1 | 9 | **YES — top five** |
| `run_19` | 69.751 | 169 | 151 | 3 | 0 | 1 | 1 | 3 | **YES — top five** |
| `run_21` | 44.568 | 112 | 96 | 2 | 0 | 1 | 1 | 3 | No |
| `run_22` | 18.087 | 39 | 21 | 4 | 0 | 1 | 1 | 3 | No |
| `run_27` | 5.263 | 22 | 7 | 1 | 0 | 1 | 1 | 3 | No |
| `run_29` | 8.992 | 26 | 11 | 1 | 0 | 1 | 1 | 3 | No |
| `run_31` | 36.444 | 359 | 321 | 4 | 0 | 2 | 2 | 10 | No |
| `run_32` | 19.749 | 43 | 25 | 4 | 0 | 1 | 1 | 3 | No |
| `run_33` | 56.744 | 137 | 114 | 4 | 0 | 1 | 1 | 7 | No |
| `run_34` | 19.366 | 158 | 120 | 4 | 0 | 2 | 2 | 10 | No |
| `run_35` | 26.066 | 287 | 121 | 5 | 123 | 2 | 2 | 14 | No |
| `run_41` | 17.418 | 50 | 24 | 2 | 0 | 1 | 1 | 13 | No |
| `run_42` | 29.323 | 268 | 231 | 3 | 0 | 2 | 2 | 10 | No |
| `run_43` | 110.386 | 360 | 319 | 8 | 0 | 2 | 2 | 8 | **YES — top five** |
| `run_44` | 3.219 | 22 | 2 | 1 | 1 | 1 | 1 | 7 | No |
| `run_45` | 2.803 | 20 | 1 | 1 | 1 | 1 | 1 | 5 | No |
| `run_46` | 0.016 | 8 | 0 | 0 | 0 | 1 | 1 | 0 | No |
| `run_47` | 152.469 | 953 | 907 | 7 | 0 | 2 | 2 | 14 | **YES — top five** |
| `run_48` | 3.219 | 24 | 2 | 1 | 1 | 1 | 1 | 9 | No |
| `run_49` | 16.772 | 173 | 141 | 2 | 0 | 2 | 2 | 6 | No |
| `run_50` | 14.517 | 212 | 154 | 3 | 6 | 2 | 2 | 24 | No |

**Path convention:** Each listed directory has absolute path `/scratch/IITB/CompnalAstrphyNRelvity/23n0315/<run_id>`, as shown in the updated inventory screenshot. The original read-only scan identifies folders matching `run_*` or `runN*` and may not represent other unlisted data under scratch.


## 4. Reproducibility information (to attach / verify)

For **each** run, document the following before marking it fully reproducible:

1. **Exact FLASH source and custom modifications**, including changed initialization, gravity, rotation and magnetic-field routines; Git commit / remote and any uncommitted source-code changes.
2. **Build recipe / execution environment:** FLASH version, build flags, compiler, MPI library, loaded modules, node count and task count.
3. **Runtime parameters and submission details:** exact `flash.par`, SLURM batch script, command used, walltime and run status.
4. **Initial stellar model:** MESA profile used, original MESA inlists/model and mapping script, plus any relaxation or restart checkpoint.
5. **Diagnostics and reduced products:** scripts and plot-ready data for central density, radial profiles, kinetic and magnetic energies, rotation and magnetic diagnostics.
6. **Data description:** FLASH HDF5 plot-file (`*_hdf5_plt_cnt_*`), checkpoint (`*_hdf5_chk_*`) and other file conventions; physical units (FLASH cgs); reason for scientific retention.

**Project-wide context only (do not assume identical settings for every run):** FLASH 4.7.1 white-dwarf simulations with Helmholtz EOS and gravitational-field handling; hydrodynamic relaxation followed by selected rotation/MHD configurations. Exact per-run settings must come from the archived `flash.par`, code revision and logs. Some work used MESA `make_co_wd` profiles. These general descriptions are insufficient on their own to reproduce a specific run.

The supplied `compastro-DataManifest.sh` can generate `PROVENANCE.txt`, `MANIFEST.sha256` and catalogue entries. According to CompAstroGroup guidelines, provenance must be recorded **before** creating the checksum manifest, and changed data require a deliberately refreshed manifest.

## 5. Transfer and integrity plan

- **Destination:** group NAS `CompAstroGroup/users/<NAS_USERNAME>/` or a professor-approved project folder (destination and user to be confirmed).
- **Connectivity:** from the Rudra login node, check whether `10.112.5.58:2222` is reachable. Use direct rsync if reachable; otherwise use the student guide's laptop SSH relay or two-hop transfer.
- **Order:** preserve code, `flash.par`, scripts, MESA initial data and this README; record provenance where possible; create checksum manifests for finished runs; then transfer the five newly selected folders **one at a time**.
- **Validation:** verify SHA-256 file manifests on accessible copies (or have the NAS admin verify) and inspect transfer logs. An rsync exit code alone is not equivalent to a complete checksum verification.
- **Safety:** no `rsync --delete`, no `chmod/chown/chgrp` on the NAS, no removal of scratch data before backup verification. For irreplaceable outputs, follow the group's multi-copy policy even after NAS transfer.
- **Post-transfer:** update this README with destination, transfer dates, checksum/verification result and any scratch deletion approval.

## 6. Transfer-status register (to update)

| Run | NAS destination | Transfer completed | Verification completed | Scratch deleted |
|---|---|---|---|---|
| `run_17` | Pending | No | No | No |
| `run_47` | Pending | No | No | No |
| `run_43` | Pending | No | No | No |
| `run_16` | Pending | No | No | No |
| `run_19` | Pending | No | No | No |

## 7. Documents and provenance sources

- Rudra read-only run scan: `~/RUDRA_RUNS_2026-10-21.tsv`, generated by `rudra_inventory.sh`.
- CompAstroGroup: `CompAstroGroup-Data-Guidelines.md`, especially §§3 (integrity), 4 (reproducibility), 5 (catalogue/priorities), 6 (reduced data), 8 (reporting).
- CompAstroGroup: `CompAstroGroup-Student-Guide.md`, especially §§2.6–2.7 (manifest + Rudra-to-NAS copying).
- Supplied tool: `compastro-DataManifest.sh` (its actual uploaded filename; change command examples in the documentation accordingly).

**Outstanding items before declaring this an archival record:** run-by-run physics descriptions, exact source/commit/build and `flash.par` information, reduced-data location, backup destination, checksum manifests, verified transfer status, and retention priorities.
