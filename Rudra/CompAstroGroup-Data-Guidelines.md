# CompAstroGroup: Data Integrity and Reproducibility Guidelines

For everyone in the group who produces or analyses research data (simulations, observations, derived products). Last updated: 2026-10-14.

Contents

- 0. Quick start: six rules
- 1. The `compastro-manifest.sh` script
- 2. Why data integrity matters
- 3. Integrity rules
- 4. Scientific reproducibility
- 5. Size estimates, run catalogue and priorities
- 6. Reduced data
- 7. Checklist for every run
- 8. Reporting your data to the group
- 9. References

---

## 0. Quick start: six rules

1. Before launching a run, estimate how much data it will write, and check it fits where it will be stored (5.1).
2. Run only from committed and pushed code; record the code commit, run command, parameter file and initial data of every run (4).
3. Make a checksum manifest for every finished dataset and verify every copy you make (3.1, 3.2).
4. Keep a second copy of anything you cannot regenerate, on different storage (3.3).
5. Keep small reduced data for every run, so the big raw data can be deleted when space is needed (6).
6. Keep a run catalogue with sizes and priorities, commit it to your project repository, and report it to the group (5, 8).

---

## 1. The `compastro-manifest.sh` script

One script handles manifests, provenance, size estimates and the run catalogue. It runs on your laptop or on a cluster login node (Linux or macOS; needs bash, git and rsync).

```
chmod +x compastro-manifest.sh
./compastro-manifest.sh help                                   # all commands
export RUN_CATALOG=~/projects/<project>/RUNS.tsv               # your run catalogue (5.2); put this in ~/.bashrc or ~/.zshrc
```

| Command | When | What it does |
|---|---|---|
| `estimate 256x256x256 8 200 [bytes] [overhead%]` | before launching | estimated output size (5.1) |
| `estimate-from TESTDIR 'out_*.h5' 500` | before launching | scale a short test run to the full run |
| `provenance RUNDIR CODEREPO FILES...` | run finished | record code commit, run command, parameters, initial data (4.2) |
| `create RUNDIR [PRIORITY]` | after provenance | write `MANIFEST.sha256` + `MANIFEST.info`, add the run to the catalogue |
| `priority RUNDIR 1-4` | any time | change the priority (5.3); add a note with `NOTE="..."` |
| `verify RUNDIR` | after any copy | re-check every file against the manifest |
| `upload RUNDIR DEST` | copying | rsync to `DEST` (e.g. `user@host:/path/` or `/media/disk/`), compare byte by byte, print `UPLOAD VERIFIED` |
| `list` | any time | all runs with sizes, priorities and totals |
| `candidates` | space needed | runs that could be deleted, largest first, and the space it would free |
| `refresh` | any time | re-measure catalogued runs present on this machine |

Typical sequence for a finished run (on the cluster login node, where the run was made):

```
RUN_CMD="mpirun -n 512 ./athena -i athinput.disk" \
./compastro-manifest.sh provenance run42 ~/codes/athenak run42/athinput.disk run42/job.slurm
./compastro-manifest.sh create   run42 2
./compastro-manifest.sh upload   run42 /path/to/second/copy/
./compastro-manifest.sh list
```

Order matters: `provenance` before `create`, so the manifest also covers the provenance files. If you add provenance or reduced data later, refresh the manifest with `FORCE=1 ./compastro-manifest.sh create run42`.

---

## 2. Why data integrity matters

Research data can be lost or silently damaged in ways nobody notices for months:

- Silent corruption ("bit rot"): a disk, cable, controller or memory error changes bytes without any error message. Large field studies found checksum mismatches across a measurable fraction of production disks (Bairavasundaram et al. 2008, Abstract and Sect. 4).
- Disk failure: real-world disk replacement rates are well above datasheet values (Schroeder and Gibson 2007, Abstract and Sect. 4).
- Bad transfers: an interrupted or flaky copy leaves a truncated or damaged file that looks normal.
- Human error: deleting the wrong folder, a script overwriting results, `rsync --delete` in the wrong direction, a purged cluster scratch directory.
- Disasters: theft, fire, power surges, ransomware, a lost laptop.

No single measure covers all of these:

| Protection | Protects against | Does NOT protect against |
|---|---|---|
| Storage with redundancy (RAID, checksumming file systems, snapshots) where available | failed disks, some silent corruption, recent deletions | your own mistakes copied everywhere; loss of the whole system |
| Second copy on different storage (3.3) | loss of one system, theft, fire, purged scratch | corruption copied into both |
| `MANIFEST.sha256` per dataset (3.1) | bad transfers, silent corruption on any copy | nothing by itself: it detects; copies restore |
| Checksums inside files (FITS, HDF5) (3.5) | damaged files being read without notice | detection only |
| README + durable formats (3.5, 3.7) | data intact but no longer understandable or readable | physical loss |
| Provenance + reduced data (4, 6) | losing raw data that can be regenerated | irreplaceable data (keep two copies) |

The key idea: manifests prove that every copy is still identical to the original; second copies give you something to restore from when a check fails; provenance and reduced data make regenerable data cheap to lose.

---

## 3. Integrity rules

### 3.1 Manifest every finished dataset

A manifest is a list of SHA-256 checksums, one per file. If even one bit of a file changes, its checksum changes.

```
./compastro-manifest.sh create  ~/sims/run42 2     # writes MANIFEST.sha256 and MANIFEST.info, priority 2
./compastro-manifest.sh verify  ~/sims/run42       # re-checks every file
```

- Make it when the dataset is FINISHED, on the machine that produced it, so the checksum describes the original.
- Do not rename or delete `MANIFEST.sha256`.
- Changed a finished dataset? Make a new version folder (`run42_v2`) or refresh on purpose with `FORCE=1 ./compastro-manifest.sh create ...`. Never let a manifest go stale silently.
- Without the script (any Linux machine):

  ```
  cd run42 && find . -type f ! -name MANIFEST.sha256 -print0 | sort -z | xargs -0 sha256sum > MANIFEST.sha256
  sha256sum -c --quiet MANIFEST.sha256 && echo OK
  ```

### 3.2 Verify after every copy

- Copy with `./compastro-manifest.sh upload RUNDIR DEST`: it copies with rsync, compares source and destination byte by byte, and prints `UPLOAD VERIFIED` only if identical.
- After downloading or copying in any other way (laptop, cluster, external disk): `./compastro-manifest.sh verify <folder>`.
- Keep the original until the copy is verified; never delete the cluster copy of a run before the second copy is verified.
- Use rsync for large data, not drag-and-drop or a browser: rsync resumes after interruptions and can verify.
- Never use `rsync --delete` unless you are sure of the direction; it deletes files at the destination.

### 3.3 Two copies of irreplaceable data: 3-2-1

- Three copies of anything you cannot regenerate (raw observations, unique simulation output, anything behind a paper), on two different kinds of storage, one of them off site.
- Cluster scratch space is NOT storage: it is usually purged and not backed up. Move finished runs off scratch promptly.
- A laptop is not storage either: it can be lost or fail. Treat laptop copies as working copies.
- Data you can regenerate (complete provenance, section 4) needs only one copy plus its reduced data; note in the README how to regenerate it.

### 3.4 Freeze finished data

- Treat finished datasets as read-only. Never edit raw data in place; write derived products into a new folder (`run42/analysis_2026-10/` or `run42/reduced/`).
- Optionally make it read-only on Linux/macOS storage you own: `chmod -R a-w run42` (undo with `chmod -R u+w run42`).

### 3.5 Durable file formats, with built-in checksums

Prefer open, self-describing formats that will still open in 10 years: FITS, HDF5, NetCDF; plain text or CSV for small tables. Avoid Python pickles, `.npy` files with object arrays, and tool-specific binary formats for anything you keep (they break when library versions change).

Turn on the checksums built into the formats, so damage is caught the moment a file is read:

```
# FITS (astropy): write CHECKSUM/DATASUM keywords, verify on open
from astropy.io import fits
hdul.writeto("map.fits", checksum=True)
with fits.open("map.fits", checksum=True) as h:   # warns if a checksum does not match
    data = h[0].data

# HDF5 (h5py): Fletcher-32 checksum per chunk; a damaged chunk raises an error on read
import h5py
with h5py.File("out_0100.h5", "w") as f:
    f.create_dataset("density", data=rho, chunks=True, fletcher32=True)
```

The FITS keywords are defined in the FITS Standard 4.0, Sect. 4.4.2.7; the HDF5 filter is described in the h5py documentation, "Filter pipeline".

### 3.6 Archives and compression

- Many small files (thousands of snapshots, logs): pack them into one `.tar` or one HDF5 file before copying; they copy and verify much faster.
- Avoid one huge `.tar.gz`: a single damaged byte can make everything after it unreadable. Prefer an uncompressed `.tar`, compress files individually, or use `zstd` (which stores its own checksums): `tar -I zstd -cf run42.tar.zst run42/`.
- Always keep the manifest (of the original files, or of the archive file itself).

### 3.7 Describe the data (README)

A bit-perfect file that nobody can interpret is still lost data. Put a `README.txt` in every dataset folder:

```
Dataset:     run42  (accretion disk, alpha=0.1, 256^3)
Project:     <project / paper, arXiv or DOI when available>
Produced by: <name>, <date YYYY-MM-DD>
Code:        AthenaK, git commit <hash>, built with <compiler/modules>
Inputs:      athinput.disk (copied into this folder), command line used
Machine:     <cluster>, <nodes x cores>, <runtime>
Contents:    out_XXXX.h5 = snapshots every 10 orbits; hst = history file
Units:       code units, G=M=1; length unit = r_g
Priority:    2 (see RUNS.tsv)
Reduced:     reduced/ts.h5 (time series), reduced/profiles.h5
Notes:       snapshots 0-49 removed (initial transient); regenerate with <command>
Manifest:    MANIFEST.sha256 (created <date> on <machine>)
```

This also covers provenance (who, how, with what), one of the FAIR principles (Wilkinson et al. 2016, Box 2, principle R1.2).

### 3.8 Names

- Letters, digits, `-`, `_`, `.` only; no spaces or special characters.
- Dates as `YYYY-MM-DD` so they sort correctly, e.g. `2026-10-12_disk-alpha0.1_run42`.

### 3.9 When a check fails

| Message | Do |
|---|---|
| `UPLOAD MISMATCH` | Delete nothing. Re-run the same upload (rsync re-sends only the bad files). Still failing: report to the group lead with the path and the output. |
| `VERIFY FAILED` on a copy | Re-copy from the original (or another verified copy) and verify again. |
| `VERIFY FAILED` on the original | Compare with your second copy; restore the damaged files from it; report to the group lead. |
| FITS checksum warning / HDF5 read error | Do not overwrite the file. Fetch a fresh copy and compare. |

---

## 4. Scientific reproducibility

Goal: anyone in the group (including you in three years) can regenerate a run bit-for-bit, or at least to the same physics, from what is recorded. This is also what allows large raw data to be deleted safely (5.3, 6). The practices below follow Sandve et al. (2013), Rules 1 (track how every result was produced), 3 (archive exact program versions), 4 (version-control all custom scripts), 6 (record random seeds) and 7 (store the data behind plots).

### 4.1 What must be recorded for every run

| Item | Where it lives | How |
|---|---|---|
| Exact code version | git commit hash of the simulation code | commit and push BEFORE launching; never run from uncommitted changes |
| Local code changes | none, ideally | if unavoidable, the diff is saved as `provenance/code.diff` (the script does this and warns) |
| Build configuration | project git repository | configure/cmake command, compiler, modules (`module list`), problem generator, flags |
| Runtime parameters | copied into the run folder and committed to the project repository | the parameter / input file exactly as used (e.g. `athinput.disk`) |
| Run command | `PROVENANCE.txt` | `RUN_CMD="mpirun -n 512 ./athena -i athinput.disk"` when running `provenance` |
| Initial data | run folder (if small) or checksum + path (if large) | plus the script that generated it |
| Instructions to create initial data | project git repository | script + its parameters + its own git hash; ideally one command, e.g. `make_ic.py --config ic.yaml` |
| Job script | project git repository and run folder | the batch script submitted (nodes, cores, walltime) |
| Analysis code for reduced data / figures | project git repository | commit hash recorded in `reduced/PROVENANCE.txt` |
| Random seeds | parameter file | never rely on time-based seeds |

Keep one project git repository per project (e.g. on gitlab.iitb.ac.in) with:

```
<project>/
  README.md        what the project is, how to build the code, how to run
  runs/run42/      parameter file, job script, notes (one folder per run)
  ic/              initial-data scripts and their configs
  analysis/        reduction and plotting scripts
  RUNS.tsv         the run catalogue (section 5)
```

### 4.2 Record it with the script

Run this on the machine where the run was made (e.g. the cluster login node), as soon as the run finishes:

```
# code repository, then every file needed to rerun: parameters, job script, initial data or its generator
RUN_CMD="mpirun -n 512 ./athena -i athinput.disk" \
./compastro-manifest.sh provenance  run42  ~/codes/athenak  \
    run42/athinput.disk  run42/job.slurm  ~/projects/disk/ic/make_ic.py  ~/projects/disk/ic/ic.yaml
```

This writes `run42/PROVENANCE.txt` (code commit, branch, remote, dirty flag, machine, loaded modules, run command, checksums) and copies the small files into `run42/provenance/`. Files over 50 MB (e.g. large initial data) are recorded by path and checksum only; give large initial data its own manifest and keep it with the run.

If the script warns about UNCOMMITTED changes: commit and push them, then run `provenance` again. A run made from a dirty working copy is only reproducible through the saved diff, which is fragile.

Then create the manifest:

```
./compastro-manifest.sh create run42 2
```

---

## 5. Size estimates, run catalogue and priorities

### 5.1 Estimate the size BEFORE launching a run

```
# grid, variables written per cell, number of output snapshots [, bytes per value (8 = double), overhead %]
./compastro-manifest.sh estimate 512x512x512 8 400
./compastro-manifest.sh estimate 512x512x512 8 400 4        # single precision output

# or measure a short test run and scale it up
./compastro-manifest.sh estimate-from ~/sims/test42 'out_*.h5' 400
```

Compare the estimate with your quota on the cluster and on the storage where the data will finally live (`quota`, `df -h <path>`, or your cluster's quota command). If it is too large:

- write fewer snapshots, or full snapshots rarely and reduced quantities (time series, slices) often;
- write single precision;
- write only the variables you need;
- add restart/checkpoint files to the estimate (often 1 to 3 snapshots each), and delete old restarts.

Note the estimate in the run's notes (`runs/run42/notes.md`) and discuss runs above a few TB with the group lead before launching.

### 5.2 The run catalogue

Every `create` adds the run to your catalogue `RUNS.tsv`, a tab-separated table with one line per run: path, name, host, date, number of files, size, priority, code commit, dirty flag, size of `reduced/`, note.

```
export RUN_CATALOG=~/projects/disk/RUNS.tsv          # keep it in the project git repository
./compastro-manifest.sh list                          # all runs by priority, sizes and totals
./compastro-manifest.sh refresh                       # re-measure runs present on this machine
```

- Use one catalogue per project; if runs live on several machines (cluster, laptop), keep one catalogue per machine (e.g. `RUNS-rudra.tsv`, `RUNS-laptop.tsv`) in the same repository.
- Commit and push the catalogue after every change.

### 5.3 Priorities

Give every run a priority when you create the manifest, and review them whenever space is tight:

| Priority | Meaning | Examples | May be deleted? |
|---|---|---|---|
| 1 | keep forever | irreplaceable data, data behind a published or submitted paper | no (keep two copies) |
| 2 | keep | runs in active analysis | not now |
| 3 | regenerable | complete provenance (clean commit, parameters, initial data) AND reduced data exists | yes, when space is needed |
| 4 | scratch | tests, failed or superseded runs | yes, any time |

```
./compastro-manifest.sh priority run42 3
NOTE="fig 4 of disk paper" ./compastro-manifest.sh priority run17 1      # with a note
./compastro-manifest.sh candidates                                       # what could be deleted, largest first
```

`candidates` lists priority 4 runs, and priority 3 runs only if they have a recorded clean commit and a non-empty `reduced/` folder, and shows how much space deleting them would free.

Before deleting:

1. Check nobody else uses the run.
2. For priority 3: confirm `PROVENANCE.txt` is complete and the reduced data is verified and stored safely.
3. Delete, then keep the line in the catalogue with a note (`NOTE="deleted 2026-12-01" ./compastro-manifest.sh priority run42 3` before deleting, or edit `RUNS.tsv`) and commit it. The record of how to regenerate the run stays.

Priority 1 data is never deleted without the group lead's agreement.

---

## 6. Reduced data: small, plot-ready, kept long

Full snapshots are rarely needed for figures and results. For every run, derive the quantities you actually use and keep them small and safe:

- Put them in `run42/reduced/` (one or a few HDF5 files with `fletcher32=True`, see 3.5): time series, volume averages, radial profiles, spectra, selected slices or projections, the exact arrays behind each figure.
- Keep the analysis scripts in the project git repository and record their version:

  ```
  ./compastro-manifest.sh provenance run42/reduced ~/projects/disk  analysis/reduce_run.py analysis/fig4.py
  ```

- Typical size: MB to a few GB instead of hundreds of GB to TB. Reduced data plus complete provenance is what makes a run priority 3 instead of 1: the raw data can then be deleted and regenerated if ever needed.
- Data behind a published figure: keep its reduced data at priority 1, with two copies, even if the raw run is deleted (Sandve et al. 2013, Rule 7).
- Refresh the manifest after adding reduced data: `FORCE=1 ./compastro-manifest.sh create run42`.

---

## 7. Checklist for every run

```
Before:   commit + push code  ->  estimate size  ->  check quota  ->  note estimate in runs/<run>/notes.md
Launch:   parameter file, job script, initial data / generator committed in the project repo; seeds fixed
After:    provenance  ->  reduced/ + its provenance  ->  create (with priority)  ->  verified second copy
Then:     list, commit RUNS.tsv, move the run off cluster scratch
Later:    review priorities; delete priority 4, and priority 3 once reduced data is safe
```

---

## 8. Reporting your data to the group

So that the group can plan storage:

- Keep `RUNS.tsv` up to date and pushed in your project repository; the group lead reads it from there.
- Once a month (and before asking for more space or compute time), run and send the summary:

  ```
  ./compastro-manifest.sh refresh
  ./compastro-manifest.sh list        # totals per priority
  ./compastro-manifest.sh candidates  # what could be freed
  ```

- Announce planned large runs (estimate from 5.1) before launching them.
- When you leave the group or finish a project: make sure every run has priority, provenance and reduced data, priority 1 data has two verified copies, and the project repository (with `RUNS.tsv`) is complete and pushed.

---

## 9. References

1. Bairavasundaram, L. N. et al., "An Analysis of Data Corruption in the Storage Stack", Proc. 6th USENIX Conference on File and Storage Technologies (FAST '08), 2008; Abstract and Sect. 4. https://www.usenix.org/conference/fast-08/analysis-data-corruption-storage-stack
2. Schroeder, B. and Gibson, G. A., "Disk failures in the real world: What does an MTTF of 1,000,000 hours mean to you?", Proc. 5th USENIX FAST (FAST '07), 2007; Abstract and Sect. 4. https://www.usenix.org/conference/fast-07/disk-failures-real-world-what-does-mttf-1000000-hours-mean-you
3. Sandve, G. K., Nekrutenko, A., Taylor, J. and Hovig, E., "Ten Simple Rules for Reproducible Computational Research", PLoS Computational Biology 9(10), e1003285 (2013), Rules 1, 3, 4, 6, 7. https://doi.org/10.1371/journal.pcbi.1003285
4. Wilkinson, M. D. et al., "The FAIR Guiding Principles for scientific data management and stewardship", Scientific Data 3, 160018 (2016), Box 2 (principle R1.2). https://doi.org/10.1038/sdata.2016.18
5. IAU FITS Working Group, "Definition of the Flexible Image Transport System (FITS), Version 4.0" (2018), Sect. 4.4.2.7 (CHECKSUM and DATASUM). https://fits.gsfc.nasa.gov/standard40/fits_standard40aa-le.pdf
6. h5py documentation, "Datasets", Filter pipeline (fletcher32). https://docs.h5py.org/en/stable/high/dataset.html
7. Astropy documentation, "Verification" (FITS checksums). https://docs.astropy.org/en/stable/io/fits/usage/verification.html
8. rsync(1) manual, sections "USAGE" and "OPTIONS" (--delete, --checksum). https://download.samba.org/pub/rsync/rsync.1
9. US-CERT / CISA, "Data Backup Options" (2012), 3-2-1 rule. https://www.cisa.gov/sites/default/files/publications/data_backup_options.pdf
