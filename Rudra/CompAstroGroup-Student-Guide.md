# CompAstroGroup NAS: Student Guide

For members of the CompAstroGroup (IIT Bombay). Last updated: 2026-10-14.

Contents

- 1. Quick start: what the NAS is, where it is, what you can do
- 2. Using the NAS (connect, transfer, share, check usage, the manifest script, cluster data, data on your laptop)
- 3. Data integrity, reproducibility and run management (with `CompAstroGroup-Data-Guidelines.md`)
- 4. Troubleshooting
- 5. References

Companion document: `CompAstroGroup-Data-Guidelines.md` (cited as DG) for data integrity, reproducibility and run management. Keep both, plus `compastro-manifest.sh`.

---

## 1. Quick start

### 1.1 What you get

- An account on the group NAS (a Synology storage server) with your own private folder `users/<you>/`, the group area `shared/`, and project folders you are added to.
- Transfer and access with rsync, sftp and the web page (File Station). There is no shell (no `ssh` login to a command prompt), no network drive (SMB) and no Synology Drive sync for student accounts.
- The admin can read all data on the NAS.

### 1.2 Where it is

| You are | Web page (File Station) | rsync / sftp host and port |
|---|---|---|
| In the lab, on the lab router network | `https://192.168.0.192:5001` | `192.168.0.192`, port 22 |
| Anywhere on campus or on the campus VPN | `https://10.112.5.58:5001` | `10.112.5.58`, port 2222 |
| Off campus without the campus VPN | not reachable | not reachable |

Accept the browser's certificate warning the first time. Never use port 22 with `10.112.5.58` (that is the router, not the NAS).

### 1.3 Folder layout

```
CompAstroGroup/
  users/<you>/          your private folder; you decide who else sees it
  projects/<project>/   one per project; you see it if you are in the project group
  shared/               whole group can read and write
```

Path to use: rsync `/volume2/CompAstroGroup/...`, sftp `/CompAstroGroup/...`.

### 1.4 Six rules

1. Never run `chmod`, `chown` or `chgrp` on NAS files (also not at the `sftp>` prompt); share only through File Station (2.4).
2. Make a checksum manifest for every finished dataset and verify every transfer (2.6; DG 3.1, 3.2).
3. Make every run reproducible: launch only from committed and pushed code, and record the code commit hash, run command, parameter file, initial data and the instructions to create it, with `compastro-manifest.sh provenance` and in your project git repository (DG 4).
4. Never use `rsync --delete` towards the NAS; deletions may not be recoverable.
5. Do not start several large transfers at once; the NAS is shared.
6. Every run: estimate its size before launching, keep reduced data, and give it a priority in your run catalogue (DG 5 to 7).

---

## 2. Using the NAS

You have no shell on the NAS (DSM gives shells only to administrators). You use rsync, sftp and the DSM web page. All commands below run on YOUR laptop (Linux or macOS; on Windows use WSL).

### 2.1 One-time setup on your laptop

Add this to `~/.ssh/config` (replace `alice` with your username):

```
Host compastro
    HostName 10.112.5.58
    Port 2222
    User alice
```

Test:

```
sftp compastro
```

Optional, password-free logins: ask the admin to install your public key (`~/.ssh/id_ed25519.pub`).

You must be on the campus network (or campus VPN) to reach the NAS.

Web page (File Station: browse, upload, download, share; quota under person icon > Personal > Quota). Log in with your own NAS username:

| Where you are | Address |
|---|---|
| In the lab, on the lab router network | `https://192.168.0.192:5001` |
| Anywhere on campus or campus VPN | `https://10.112.5.58:5001` |
| Off campus without campus VPN | not reachable |

Accept the browser's certificate warning the first time.

### 2.2 Upload and download with rsync

```
# upload a folder (note: no trailing slash on the source = copy the folder itself)
rsync -rltvhP ~/sims/run42 compastro:/volume2/CompAstroGroup/users/alice/

# download
rsync -rltvhP compastro:/volume2/CompAstroGroup/users/alice/run42 ~/Downloads/

# put something in the group area
rsync -rltvhP catalog.fits compastro:/volume2/CompAstroGroup/shared/

# preview first (copies nothing)
rsync -rltvhPn ~/sims/run42 compastro:/volume2/CompAstroGroup/users/alice/
```

Flags: `-r` recurse, `-l` keep symlinks, `-t` keep timestamps (re-runs skip unchanged files), `-v` verbose, `-h` human sizes, `-P` progress + resume, `-n` dry run. An interrupted transfer: just run the same command again.

Do not use `--delete` unless you understand it: it deletes files on the destination that are not in the source.

Without the ssh config entry, add `-e "ssh -p 2222"` and use `alice@10.112.5.58:` instead of `compastro:`.

### 2.3 Interactive file browsing with sftp

```
sftp compastro
sftp> ls
sftp> cd /CompAstroGroup/users/alice
sftp> put results.h5
sftp> get -r run42
sftp> df -h          # free space on the volume
sftp> ls -lh         # file sizes
sftp> quit
```

### 2.4 Sharing your files: read-only for some, read/write for others

Your folder `users/<you>` is private by default: only you and the admin can open it. You decide who sees what, item by item. Sharing is done in File Station (DSM web page, `https://10.112.5.58:5001` from campus, `https://192.168.0.192:5001` in the lab). It cannot be done with sftp, rsync or `chmod`.

Do NOT use `chmod`, `chown` or `chgrp` on NAS files, including at the `sftp>` prompt: SFTP only knows Unix permissions; on the NAS they replace the Synology permissions of that item, which can lock you out, break inherited sharing or expose files. Permissions can only be changed in File Station (web page, from campus or lab), as described below.

#### Tip: organise by sharing level first

Sharing whole folders is far easier than sharing single files. A layout like this keeps it simple:

```
users/alice/
  private/        nobody else
  share-read/     colleagues may read
  share-rw/       colleagues may read and write
```

#### Example: bob may READ `run42/` and READ+WRITE `paper-draft/`

All steps in File Station, logged in as yourself (alice).

Step 1. Let bob enter your folder (once per colleague):

1. Open `CompAstroGroup/users`, right-click `alice` > Properties > Permission > Create.
2. User or group: `bob`. Type: Allow.
3. Apply to: tick ONLY "This folder" (untick Child folders, Child files, All descendants).
4. Permission: Read > tick Traverse folders/Execute files, List folders/Read data, Read attributes, Read extended attributes, Read permissions. Nothing under Write or Administration.
5. Done > Save.

Without this step bob cannot reach anything inside your folder. He still cannot read anything else of yours: items you have not shared stay hidden from him.

Step 2. Read-only access to `run42/`:

1. Right-click `run42` > Properties > Permission > Create.
2. User or group: `bob`, Type: Allow, Apply to: All (all four boxes).
3. Permission: tick the whole Read group. Nothing under Write or Administration.
4. Done > Save.

Step 3. Read/write access to `paper-draft/`:

1. Right-click `paper-draft` > Properties > Permission > Create.
2. User or group: `bob`, Type: Allow, Apply to: All.
3. Permission: tick the whole Read group and the whole Write group. Nothing under Administration.
   - Want bob to add and edit files but never delete them? Under Write, untick "Delete subfolders and files" and "Delete".
4. Done > Save.

A single file works the same way: right-click the file instead of a folder (Apply to is then not needed).

#### Sharing a whole folder tree (folder and all its subfolders, nothing else)

Permissions set with "Apply to: All" flow down to everything inside the folder, including files and subfolders created later. Folders next to it are not affected. Example: bob gets read/write on `users/alice/projectX/` and everything below it, and nothing else of alice's.

1. Step 1 above on `users/alice` ("This folder" only, Read), so bob can reach `projectX`.
2. Right-click `projectX` > Properties > Permission > Create: `bob`, Allow, Apply to: All, whole Read group + whole Write group, nothing under Administration. Save; if offered, tick "Apply to this folder, sub-folders and files" so files already inside get it too.
3. Different levels in the same tree: add several entries on `projectX`, for example `CompAstroGroupIITB` with Read only and `bob` with Read + Write.
4. Keep one subfolder private inside a shared tree: right-click `projectX/private-notes` > Properties > Permission > Advanced options > Exclude inherited permissions (then re-add your own entry if it disappears).

#### Sharing with the whole group

Choose `CompAstroGroupIITB` instead of `bob` in steps 1 to 3. Every current and future member gets that access.

#### Rules

| Do | Do not |
|---|---|
| Give Read, or Read + Write | Give "Change permissions" or "Take ownership" to anyone (they could then lock you out or re-share your data) |
| Use folders as sharing units | Share your whole `users/<you>` folder with "Apply to: All" unless you mean everything |
| Check with Properties > Permission > Permission Inspector (choose bob) | Rely on "they will not look": un-shared means un-shared |

#### Stop sharing

Right-click the item > Properties > Permission > select bob's row > Delete > Save. Also remove his "This folder" entry on `users/alice` if he should no longer enter your folder at all.

#### How bob uses what you shared

```
rsync -rltvhP compastro:/volume2/CompAstroGroup/users/alice/run42 ~/      # copy alice's run42
sftp compastro    then:  cd /CompAstroGroup/users/alice/paper-draft
```

The admin can always read all data on the NAS.

### 2.5 Checking size and usage

| What | How |
|---|---|
| Your total usage and quota | DSM web > person icon (top right) > Personal > Quota |
| Size of one folder | File Station > right-click folder > Properties |
| Total size and file count, from the terminal | `rsync -rn --stats compastro:/volume2/CompAstroGroup/users/alice/ /tmp/sizecheck/ \| grep -E "Number of files\|Total file size"` |
| Free space on the volume | `sftp compastro` then `df -h` |

### 2.6 The `compastro-manifest.sh` script (manifests, provenance, run catalogue)

One script handles integrity, provenance and bookkeeping. Run it on your laptop or on the cluster login node (it needs bash, git, rsync; works on Linux and macOS). Background and rules are in `CompAstroGroup-Data-Guidelines.md` (section 3).

```
chmod +x compastro-manifest.sh
./compastro-manifest.sh help                                   # all commands
export RUN_CATALOG=~/projects/<project>/RUNS.tsv               # where your run list lives (DG 5.2)
```

| Command | When | What it does |
|---|---|---|
| `estimate 256x256x256 8 200` | before launching | estimated output size (DG 5.1) |
| `estimate-from TESTDIR 'out_*.h5' 500` | before launching | scale a short test run |
| `provenance RUNDIR CODEREPO FILES...` | run finished | record git hash, run command, parameters, initial data (DG 4.2) |
| `create RUNDIR [PRIORITY]` | after provenance | write `MANIFEST.sha256`, add the run to the catalogue |
| `priority RUNDIR 1-4` | any time | change priority (DG 5.3) |
| `upload RUNDIR compastro:/volume2/CompAstroGroup/users/<you>/` | transfer | rsync + byte-by-byte check, `UPLOAD VERIFIED` |
| `verify RUNDIR` | after any copy | re-check every file |
| `list` | any time | all runs with sizes, priorities, totals |
| `candidates` | space needed | runs that could be deleted, largest first |
| `refresh` | any time | re-measure catalogued runs on this machine |

Typical sequence for a finished run:

```
RUN_CMD="mpirun -n 512 ./athena -i athinput.disk" ./compastro-manifest.sh provenance run42 ~/codes/athenak run42/athinput.disk run42/job.slurm
./compastro-manifest.sh create   run42 2
./compastro-manifest.sh upload   run42 compastro:/volume2/CompAstroGroup/users/alice/
./compastro-manifest.sh list
```

Without the ssh config entry, run `export RSYNC_RSH="ssh -p 2222"` first and use `alice@10.112.5.58:` as destination.

### 2.7 Sending data from the computing cluster (Param Rudra)

You cannot log in to the NAS and pull data, so a transfer from the cluster is always started on the cluster or on your laptop. Use the first method that works for you.

#### Method 1: push directly from the Rudra login node

First check whether Rudra can reach the NAS (on the login node, not inside a job):

```
timeout 5 bash -c '</dev/tcp/10.112.5.58/2222' && echo OPEN || echo BLOCKED
```

If OPEN, send data straight from Rudra (enter your NAS password when asked):

```
rsync -rltvhP -e "ssh -p 2222" /path/on/rudra/run42 alice@10.112.5.58:/volume2/CompAstroGroup/users/alice/
```

If BLOCKED, use Method 2. Compute nodes usually have no outside network at all, so never transfer from inside a batch job.

#### Method 2: relay through your laptop (reverse ssh tunnel)

How it works:

```
Rudra login node                 your laptop (on campus)                NAS
  rsync -> localhost:22517 ===ssh tunnel===> laptop -> 10.112.5.58:2222 -> NAS
```

Your laptop's ssh session to Rudra opens a temporary port (22517 in the example) on the Rudra login node. Whatever is sent to that port travels back through your encrypted ssh connection to your laptop, and the laptop passes it on to the NAS. Rudra never has to reach the NAS itself.

What you need:

| Item | Answer |
|---|---|
| Disk space on the laptop | None. Data streams through the laptop's memory and is never stored there. Only your NAS quota must have room. |
| Laptop location | On the campus network (it must reach 10.112.5.58). |
| Time | The laptop must stay on, awake and connected for the whole transfer. Speed is limited by the laptop's network link: roughly 40 to 100 MB/s on wired gigabit (100 GB in about 20 to 45 min), 10 to 30 MB/s on Wi-Fi (100 GB in about 1 to 3 hours). |
| Passwords | Rudra password + MFA (when opening the tunnel, typed on the laptop) and your NAS password (when rsync runs, typed in the same Rudra window). Your laptop password is never needed. |

Steps (replace `alice` with your username; pick your OWN port number between 22000 and 29999 instead of 22517, because the login node is shared and two people cannot use the same port):

1. Keep the laptop awake (macOS: run `caffeinate -i` in a separate terminal; Linux: disable suspend). Prefer a wired connection for large transfers.
2. On the laptop, open the tunnel and log in to Rudra:

   ```
   ssh -p 4422 -o ServerAliveInterval=60 -R 22517:10.112.5.58:2222 alice@paramrudra.iitb.ac.in
   ```

   Enter your Rudra password and MFA. You are now in a normal Rudra shell.
   If you see `Warning: remote port forwarding failed for listen port 22517`, choose another number; if it still fails, Rudra forbids tunnels: use Method 3.
3. In that same Rudra session, send the data:

   ```
   rsync -rltvhP -e "ssh -p 22517" /path/on/rudra/run42 alice@localhost:/volume2/CompAstroGroup/users/alice/
   ```

   The first time, answer `yes` to "authenticity of host '[localhost]:22517' can't be established". Then enter your NAS password.
4. If the connection drops, open the tunnel again (step 2) and re-run the same rsync: it continues where it stopped.
5. When finished, type `exit`. Closing the session removes the tunnel. Do not leave it open for days: other Rudra users on the same login node could reach the port (they would still need your NAS password).
6. Verify: if you made a manifest on Rudra (tip below), the admin's monthly check verifies it on the NAS; to check sooner, ask the admin, or download a copy later and run `./compastro-manifest.sh verify`.

Tip: record provenance and create the manifest on Rudra before sending (copy `compastro-manifest.sh` to Rudra), so both travel with the data:

```
RUN_CMD="mpirun -n 512 ./athena -i athinput.disk" \
./compastro-manifest.sh provenance run42 ~/codes/athenak run42/athinput.disk run42/job.slurm
./compastro-manifest.sh create run42 2
```

#### Method 3: two hops through the laptop (always works, needs laptop disk space)

```
# 1. Rudra -> laptop (Rudra password + MFA)
rsync -rltvhP -e "ssh -p 4422" alice@paramrudra.iitb.ac.in:/path/on/rudra/run42 ~/transfer/

# 2. laptop -> NAS, with manifest and verification (NAS password)
./compastro-manifest.sh upload ~/transfer/run42 compastro:/volume2/CompAstroGroup/users/alice/
```

Needs free space on the laptop equal to the dataset size; delete `~/transfer/run42` after `UPLOAD VERIFIED`.

#### Method 4: very large runs, overnight from your desktop or laptop

For runs of hundreds of GB or more, start the transfer in the evening on a machine on campus that stays on overnight (a wired desktop is best), inside `tmux` so it keeps running when you close the terminal.

One-time setup on that machine:

```
# tmux: Linux: sudo apt install tmux    macOS: brew install tmux
# answer the Rudra password + MFA only once per evening (shared connection), in ~/.ssh/config:
Host rudra
    HostName paramrudra.iitb.ac.in
    Port 4422
    User alice
    ControlMaster auto
    ControlPath ~/.ssh/cm-%r@%h:%p
    ControlPersist 12h
    ServerAliveInterval 60
```

Option A: the machine has enough free disk space (two hops, with full manifest check):

```
ssh -fN rudra                         # Rudra password + MFA once
tmux new -s xfer                      # start a session that survives closing the terminal
mkdir -p ~/transfer
rsync -rltvhP rudra:/path/on/rudra/run42 ~/transfer/ \
  && ./compastro-manifest.sh upload ~/transfer/run42 compastro:/volume2/CompAstroGroup/users/alice/ \
  2>&1 | tee ~/transfer/run42.log
# enter the NAS password if asked, then detach: Ctrl-b then d
```

Option B: not enough disk space (relay through the machine, Method 2, nothing stored locally):

```
tmux new -s xfer
ssh -p 4422 -o ServerAliveInterval=60 -R 22517:10.112.5.58:2222 alice@paramrudra.iitb.ac.in
# on Rudra:
rsync -rltvhP -e "ssh -p 22517" /path/on/rudra/run42 alice@localhost:/volume2/CompAstroGroup/users/alice/ 2>&1 | tee ~/run42-xfer.log
# enter the NAS password, then detach: Ctrl-b then d
```

Keep the machine awake all night (it must not sleep or log out):

```
caffeinate -dimsu &                                    # macOS
systemd-inhibit --what=idle:sleep sleep 14h &          # Linux
```

Next morning:

```
tmux attach -t xfer                   # look at the end of the output
tail ~/transfer/run42.log             # Option A: must end with UPLOAD VERIFIED
```

- If it stopped (network drop, reboot): run the same commands again; rsync continues where it stopped.
- Option A: after `UPLOAD VERIFIED`, delete `~/transfer/run42` to free the space.
- Option B has no byte-by-byte check at the end: make the manifest on Rudra before sending (2.6, tip above), so it travels with the data and is checked on the NAS; or download a copy later and run `./compastro-manifest.sh verify`.
- Password-free NAS logins make unattended transfers easier: ask the admin to install your public key (2.1).

### 2.8 Getting data onto your laptop for analysis (and results back)

Available to students: rsync, sftp, File Station (web). Not available: SMB network drives, Synology Drive sync. The general rule: copy the part of the data you are analysing to your laptop's own disk, analyse locally (fast), and send results back. Reading files over the network inside a notebook is slow.

Use the `compastro` entry from 2.1 (or replace `compastro:` with `alice@10.112.5.58:` and add `-e "ssh -p 2222"`).

#### Method 1 (recommended): rsync a working copy, keeping the NAS folder structure

```
# bring one dataset to the laptop; -R keeps the path below the "./" marker
mkdir -p ~/nas-local
rsync -rltvhPR compastro:/volume2/CompAstroGroup/./users/alice/run42 ~/nas-local/
#   -> ~/nas-local/users/alice/run42/...

# only some files (e.g. snapshots 100 to 199 and the parameter file)
rsync -rltvhPR --include='*/' --include='out_01??.h5' --include='*.par' --exclude='*' \
  compastro:/volume2/CompAstroGroup/./users/alice/run42 ~/nas-local/

# preview first (copies nothing)
rsync -rltvhPRn compastro:/volume2/CompAstroGroup/./users/alice/run42 ~/nas-local/
```

Re-running the same command later copies only new or changed files.

In JupyterLab:

```
base = "/Users/alice/nas-local/users/alice/run42"     # Linux: /home/alice/nas-local/...
```

Send results back to the NAS (into your own folder):

```
rsync -rltvhP ~/nas-local/users/alice/run42/analysis compastro:/volume2/CompAstroGroup/users/alice/run42/
```

What is on the laptop and how big:

```
du -sh ~/nas-local                      # total
du -sh ~/nas-local/users/alice/*        # per dataset
```

Free laptop space when done (NAS copy is not affected):

```
rm -rf ~/nas-local/users/alice/run42
```

Only ever delete inside `~/nas-local`; never use `rsync --delete` towards the NAS.

#### Method 2: sftp for a few files

```
sftp compastro
sftp> cd /CompAstroGroup/users/alice/run42
sftp> lcd ~/nas-local
sftp> get out_0100.h5           # one file
sftp> get -r plots              # a folder
sftp> put results.csv           # upload
sftp> quit
```

Note the path difference: sftp uses `/CompAstroGroup/...`, rsync uses `/volume2/CompAstroGroup/...`.

#### Method 3: File Station in the browser (small amounts, no terminal)

Web page from 2.1 > File Station > select files > Download (folders are downloaded as a zip) or Upload. Fine for a few hundred MB; for GB-scale data use rsync (it resumes after interruptions, the browser does not).

#### Method 4: data from the computing cluster

See 2.7 (direct push, laptop relay, or two hops).

#### Speed to expect

Transfers are encrypted, which costs NAS CPU. Roughly 40 to 100 MB/s on a wired campus connection, 10 to 30 MB/s on Wi-Fi (100 GB: about 20 to 45 min wired, 1 to 3 hours on Wi-Fi). Once copied, the laptop SSD reads at 1000 MB/s or more, so copying first pays off whenever you read the data more than once.

Tips:

- Use a cable for large transfers; keep the laptop awake.
- Many small files transfer slowly: pack them into one HDF5 file or a `.tar` on the producing machine.
- Do not start several big transfers at once; the NAS is shared by the whole group.

---

## 3. Data integrity, reproducibility and run management

Follow `CompAstroGroup-Data-Guidelines.md` (DG below): integrity rules (DG 2, 3), scientific reproducibility (DG 4), size estimates, run catalogue and priorities (DG 5), reduced data (DG 6), the per-run checklist (DG 7) and reporting your data (DG 8). The script commands are summarised in 2.6.

What is specific to the NAS:

- What the NAS adds: redundant disks (RAID), Btrfs data checksums with regular scrubbing, a UPS with clean shutdown, daily snapshots, and a recycle bin for deletions made in File Station. These protect the NAS copy; for irreplaceable data still keep one more copy on different storage (DG 3.3).
- Copy to the NAS with `./compastro-manifest.sh upload RUNDIR compastro:/volume2/CompAstroGroup/users/<you>/`; `UPLOAD VERIFIED` is your proof that the NAS copy is identical.
- The admin re-verifies every `MANIFEST.sha256` on the NAS once a month and contacts you if a dataset fails; never rename or delete `MANIFEST.sha256`.
- Deleting on the NAS (File Station or `sftp> rm`): deletions via rsync or sftp may not go to the recycle bin. If you deleted something by mistake, tell the admin at once; recent snapshots can often bring it back.
- Before deleting priority 3 or 4 runs from the NAS (`./compastro-manifest.sh candidates`, DG 5.3), make sure nobody else uses them, and keep the catalogue line with a note.
- A check failing on the NAS copy, or a FITS/HDF5 checksum error on a file read from the NAS: tell the admin with the path and the output; do not overwrite anything.

---

## 4. Troubleshooting

| Symptom | Likely cause | What to do |
|---|---|---|
| `ssh ... -p 22 10.112.5.58` says "no matching key exchange method" | port 22 on 10.112.5.58 is the router | use port 2222 (from campus) |
| `ssh compastro` logs in and closes immediately | students have no shell | expected; use rsync / sftp / web page |
| "Connection refused" or timeout | not on campus / campus VPN, or NAS down | check `nc -vz 10.112.5.58 2222`; tell the admin if it fails on campus |
| Password correct but every login fails after several typos | your IP was auto-blocked (10 failures in 5 minutes) | wait, then ask the admin to unblock your IP |
| "Permission denied" writing into `users/` or `projects/` directly | by design: only your own folder and `shared/` are writable | write into `users/<you>/` |
| Cannot see a colleague's folder | not shared with you | ask them to share it (2.4) |
| `UPLOAD MISMATCH` / `VERIFY FAILED` | damaged or incomplete transfer | see DG 3.9 and section 3 |
| Quota exceeded | your quota is full | check usage (2.5); clean up or ask the admin |

When asking the admin for help, send: what you ran (exact command), where you were (lab / campus / VPN), and the full output.

---

## 5. References

1. rsync(1) manual, sections "USAGE" and "OPTIONS" (-r -l -t -P -n -R --include/--exclude --delete). https://download.samba.org/pub/rsync/rsync.1
2. Synology, "How can I manage ACL settings on my Synology NAS?" (sharing permissions in File Station). https://www.synology.com/hu-hu/knowledgebase/tutorials/630

Data integrity and reproducibility references: see `CompAstroGroup-Data-Guidelines.md`, section 9.
