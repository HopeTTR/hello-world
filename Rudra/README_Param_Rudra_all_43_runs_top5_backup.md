# Param Rudra — FLASH white-dwarf simulation data README

**Prepared for:** Prof. Rahul Kashyap / CompAstroGroup, IIT Bombay  
**Research area:** Computational astrophysics; numerical evolution of white dwarfs using FLASH (hydrodynamics, rotation and magnetohydrodynamics). MESA stellar models are used for initial conditions.  
**Research user:** `23n0315`  
**Inventory date:** 2026-10-16 (as recorded in `RUDRA_RUNS_2026-10-16.tsv`)  
**README date:** 2026-10-20  
**Scratch root:** `/scratch/IITB/CompnalAstrphyNRelvity/23n0315`  
**Non-scratch home directory:** `/home/IITB/CompnalAstrphyNRelvity/23n0315/`  
**Status:** Inventory complete for named run directories; **backup planned, not yet confirmed**.

## 1. Summary and storage request

- A read-only inventory identified **43 run directories** (`run_*` / `runN*`, discovery depth 3), reporting a **combined 1,315.15 GiB ≈ 1.28 TiB** in the inventory summary. `du -sh` for the entire scratch root reports **1.3T**.
- For this storage request, I would like to transfer **only the five largest run directories**: `run_47`, `run_17`, `run_43`, `run_16`, and `run_19`.
- Together these five occupy **740.450 GiB ≈ 0.723 TiB**, about **56.3%** of the inventoried space. The other 38 run directories total approximately **574.70 GiB** and **are not part of this transfer request**.
- Transfer is intended to protect these datasets and, **after a verified backup and approval to remove scratch copies**, make room for new simulations. **No deletion is authorized or documented by this README.**
- The five were selected **by size**, not by their final scientific retention priority. Scientific priority and completeness still require examination of each run's inputs and results.
- Please confirm the appropriate NAS/project destination and whether there is sufficient free storage for approximately **741 GiB plus metadata/overhead**.

## 2. Five run directories proposed for backup

| Run | Size (GiB) | Total files | FLASH plot files | FLASH checkpoints | Current action |
|---|---:|---:|---:|---:|---|
| `run_47` | 194.697 | 1,710 | 907 | 14 | Transfer requested; unverified |
| `run_17` | 182.057 | 381 | 338 | 18 | Transfer requested; unverified |
| `run_43` | 140.979 | 914 | 319 | 13 | Transfer requested; unverified |
| `run_16` | 124.058 | 1,430 | 690 | 19 | Transfer requested; unverified |
| `run_19` | 98.659 | 926 | 151 | 5 | Transfer requested; unverified |
| **TOTAL** | **740.450** | **5,361** | **2,405** | **69** | **Not yet transferred** |

All counts above are filename-pattern counts from the local run inventory and **do not establish scientific completeness or file integrity**. The exact paths are `/scratch/IITB/CompnalAstrphyNRelvity/23n0315/<run_id>`.

## 3. All 43 run directories (2026-10-16 inventory)

The table preserves the current inventory of all identified simulation folders, including the 38 **not selected** for transfer.

| Run directory | Size (GiB) | Transfer request | Scientific purpose / retention priority |
|---|---:|---|---|
| `run02` | 12.840 | No (this request) | To verify from parameters and logs |
| `run03` | 1.200 | No (this request) | To verify from parameters and logs |
| `run_08` | 22.792 | No (this request) | To verify from parameters and logs |
| `run_09` | 9.104 | No (this request) | To verify from parameters and logs |
| `run_10` | 8.561 | No (this request) | To verify from parameters and logs |
| `run_11` | 6.650 | No (this request) | To verify from parameters and logs |
| `run_12` | 11.882 | No (this request) | To verify from parameters and logs |
| `run_13` | 12.227 | No (this request) | To verify from parameters and logs |
| `run_14` | 31.067 | No (this request) | To verify from parameters and logs |
| `run_15` | 5.478 | No (this request) | To verify from parameters and logs |
| `run_16` | 124.058 | **YES — top 5** | To verify from parameters and logs |
| `run_17` | 182.057 | **YES — top 5** | To verify from parameters and logs |
| `run_18` | 0.019 | No (this request) | To verify from parameters and logs |
| `run_19` | 98.659 | **YES — top 5** | To verify from parameters and logs |
| `run_21` | 44.568 | No (this request) | Differential-rotation run in previous research notes; verify current folder contents |
| `run_22` | 18.087 | No (this request) | To verify from parameters and logs |
| `run_23` | 0.016 | No (this request) | To verify from parameters and logs |
| `run_24` | 73.229 | No (this request) | To verify from parameters and logs |
| `run_25` | 6.375 | No (this request) | To verify from parameters and logs |
| `run_26` | 13.569 | No (this request) | To verify from parameters and logs |
| `run_27` | 5.263 | No (this request) | To verify from parameters and logs |
| `run_28` | 2.770 | No (this request) | To verify from parameters and logs |
| `run_29` | 8.992 | No (this request) | To verify from parameters and logs |
| `run_31` | 36.444 | No (this request) | To verify from parameters and logs |
| `run_32` | 19.749 | No (this request) | To verify from parameters and logs |
| `run_33` | 56.744 | No (this request) | To verify from parameters and logs |
| `run_34` | 19.366 | No (this request) | To verify from parameters and logs |
| `run_35` | 26.066 | No (this request) | To verify from parameters and logs |
| `run_36` | 3.220 | No (this request) | Weak-field magnetic test in previous research notes; verify current folder contents |
| `run_37` | 1.230 | No (this request) | To verify from parameters and logs |
| `run_38` | 3.219 | No (this request) | To verify from parameters and logs |
| `run_39` | 2.803 | No (this request) | To verify from parameters and logs |
| `run_40` | 3.219 | No (this request) | To verify from parameters and logs |
| `run_41` | 17.418 | No (this request) | To verify from parameters and logs |
| `run_42` | 35.293 | No (this request) | To verify from parameters and logs |
| `run_43` | 140.979 | **YES — top 5** | To verify from parameters and logs |
| `run_44` | 3.219 | No (this request) | To verify from parameters and logs |
| `run_45` | 2.803 | No (this request) | To verify from parameters and logs |
| `run_46` | 0.016 | No (this request) | To verify from parameters and logs |
| `run_47` | 194.697 | **YES — top 5** | To verify from parameters and logs |
| `run_48` | 3.219 | No (this request) | To verify from parameters and logs |
| `run_49` | 21.426 | No (this request) | To verify from parameters and logs |
| `run_50` | 24.552 | No (this request) | To verify from parameters and logs |

**Scope notes:** All 43 paths listed in the terminal output are immediately under `/scratch/IITB/CompnalAstrphyNRelvity/23n0315/` (no nesting evident in the displayed list). The inventory searched only directories following the naming pattern; unnamed/raw data elsewhere in scratch may not be represented. The difference between the summed run sizes and `du -sh` is within the rounded display reported. The inventory file with counts and complete paths is located at `~/RUDRA_RUNS_2026-10-16.tsv` on Rudra and should be saved outside scratch and committed to the project repository.

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
- **Order:** preserve code, `flash.par`, scripts, MESA initial data and this README; record provenance where possible; create checksum manifests for finished runs; then transfer the five selected folders **one at a time**.
- **Validation:** verify SHA-256 file manifests on accessible copies (or have the NAS admin verify) and inspect transfer logs. An rsync exit code alone is not equivalent to a complete checksum verification.
- **Safety:** no `rsync --delete`, no `chmod/chown/chgrp` on the NAS, no removal of scratch data before backup verification. For irreplaceable outputs, follow the group's multi-copy policy even after NAS transfer.
- **Post-transfer:** update this README with destination, transfer dates, checksum/verification result and any scratch deletion approval.

## 6. Transfer-status register (to update)

| Run | NAS destination | Transfer completed | Verification completed | Scratch deleted |
|---|---|---|---|---|
| `run_47` | Pending | No | No | No |
| `run_17` | Pending | No | No | No |
| `run_43` | Pending | No | No | No |
| `run_16` | Pending | No | No | No |
| `run_19` | Pending | No | No | No |

## 7. Documents and provenance sources

- Rudra read-only run scan: `~/RUDRA_RUNS_2026-10-16.tsv`, generated by `rudra_inventory.sh`.
- CompAstroGroup: `CompAstroGroup-Data-Guidelines.md`, especially §§3 (integrity), 4 (reproducibility), 5 (catalogue/priorities), 6 (reduced data), 8 (reporting).
- CompAstroGroup: `CompAstroGroup-Student-Guide.md`, especially §§2.6–2.7 (manifest + Rudra-to-NAS copying).
- Supplied tool: `compastro-DataManifest.sh` (its actual uploaded filename; change command examples in the documentation accordingly).

**Outstanding items before declaring this an archival record:** run-by-run physics descriptions, exact source/commit/build and `flash.par` information, reduced-data location, backup destination, checksum manifests, verified transfer status, and retention priorities.
