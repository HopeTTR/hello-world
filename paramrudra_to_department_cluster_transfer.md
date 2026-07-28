# Transfer a Folder from PARAM Rudra to the Department Cluster

## Cluster Login Commands

### PARAM Rudra

```bash
ssh -p 4422 23n0315@paramrudra.iitb.ac.in
```

### Department Cluster

```bash
ssh 23n0315@10.112.50.10
```

## Important Result

PARAM Rudra cannot directly connect to the department cluster:

```bash
ssh 23n0315@10.112.50.10
```

This connection times out.

Therefore, the transfer must be started from the **department cluster**. The department cluster pulls the folder from PARAM Rudra.

---

## Example Paths

Source folder on PARAM Rudra:

```text
/scratch/IITB/CompnalAstrphyNRelvity/23n0315/run07
```

Destination folder on the department cluster:

```text
~/flash_runs/rudra_runs/run07
```

---

## Step 1: Check the Source Folder Size

Run this from the department cluster:

```bash
ssh -p 4422 23n0315@paramrudra.iitb.ac.in \
'du -sh /scratch/IITB/CompnalAstrphyNRelvity/23n0315/run07'
```

For `run07`, the size was approximately:

```text
248G
```

---

## Step 2: Check Free Space

On the department cluster:

```bash
df -h .
```

Make sure the available space is larger than the source folder.

---

## Step 3: Create the Destination Folder

```bash
mkdir -p ~/flash_runs/rudra_runs/run07
```

---

## Step 4: Create a Persistent SSH Connection

Open **Terminal 1** and log in to the department cluster:

```bash
ssh 23n0315@10.112.50.10
```

Remove any old SSH socket:

```bash
rm -f ~/.ssh/paramrudra_socket
```

Create the persistent connection:

```bash
ssh -M -S ~/.ssh/paramrudra_socket \
-p 4422 23n0315@paramrudra.iitb.ac.in
```

Complete the:

1. Captcha
2. Verification code
3. Password

Keep this terminal open while the transfer is running.

---

## Step 5: Test the Persistent Connection

Open **Terminal 2** and log in to the department cluster:

```bash
ssh 23n0315@10.112.50.10
```

Test the SSH socket:

```bash
ssh -S ~/.ssh/paramrudra_socket \
-p 4422 23n0315@paramrudra.iitb.ac.in hostname
```

Expected output:

```text
login01
```

The login-node number may be different.

---

## Step 6: Transfer the Folder

Run this in Terminal 2:

```bash
rsync -avhP \
-e "ssh -S ~/.ssh/paramrudra_socket -p 4422" \
23n0315@paramrudra.iitb.ac.in:/scratch/IITB/CompnalAstrphyNRelvity/23n0315/run07/ \
~/flash_runs/rudra_runs/run07/
```

### Rsync Options

- `-a`: Preserve directory structure, timestamps, and permissions
- `-v`: Show file names
- `-h`: Display human-readable file sizes
- `-P`: Show progress and retain partially transferred files

The trailing slash after `run07/` means that the contents of the source folder are copied into the destination folder.

---

## If the Connection Breaks

Possible errors:

```text
rsync: connection unexpectedly closed
Broken pipe
```

Completed files remain safe.

To continue:

1. Recreate the persistent SSH connection in Terminal 1.
2. Run the same `rsync` command again in Terminal 2.

Rsync will skip completed files and resume partially transferred files.

---

## Verify the Transfer

Check the destination folder size:

```bash
du -sh ~/flash_runs/rudra_runs/run07
```

Check the number of files:

```bash
find ~/flash_runs/rudra_runs/run07 -type f | wc -l
```

Run a final dry-run verification:

```bash
rsync -avhn \
-e "ssh -S ~/.ssh/paramrudra_socket -p 4422" \
23n0315@paramrudra.iitb.ac.in:/scratch/IITB/CompnalAstrphyNRelvity/23n0315/run07/ \
~/flash_runs/rudra_runs/run07/
```

If no files are listed for transfer, the source and destination are synchronized.

---

## Final Workflow

```text
Laptop
   |
   v
Department Cluster
   |
   | rsync pull
   v
PARAM Rudra Scratch Folder
```

## Main Rule

> Run `rsync` from the department cluster, not from PARAM Rudra.
