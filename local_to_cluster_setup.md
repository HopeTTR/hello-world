# Param Rudra + VS Code Quick Setup Guide

**Author:** Rashmi Meena
**Date:** June 2026

---

## Goal

Use a Windows desktop to work remotely on the Param Rudra cluster using SSH and VS Code.

---

## 1. Verify OpenSSH on Windows

Check whether OpenSSH is installed:

```powershell
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH.Client*'
```

Verify SSH works:

```powershell
ssh
```

---

## 2. Generate SSH Keys

Generate an SSH key pair:

```powershell
ssh-keygen -t ed25519
```

Accept all default options.

Files created:

```text
C:\Users\ADMIN\.ssh\id_ed25519
C:\Users\ADMIN\.ssh\id_ed25519.pub
```

* `id_ed25519` → Private key (keep secret)
* `id_ed25519.pub` → Public key

---

## 3. Enable Passwordless Login

Display the public key:

```powershell
type $HOME\.ssh\id_ed25519.pub
```

Copy the entire output.

Login to Param Rudra:

```bash
ssh 23n0315@paramrudra.iitb.ac.in -p 4422
```

Add the public key to:

```bash
~/.ssh/authorized_keys
```

Set permissions:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

Test:

```powershell
ssh 23n0315@paramrudra.iitb.ac.in -p 4422
```

If no password is requested, setup is successful.

---

## 4. Create SSH Shortcut

Create:

```text
C:\Users\ADMIN\.ssh\config
```

Contents:

```text
Host paramrudra
    HostName paramrudra.iitb.ac.in
    User 23n0315
    Port 4422
    IdentityFile ~/.ssh/id_ed25519
```

Now login using:

```bash
ssh paramrudra
```

---

## 5. VS Code Setup

Install:

* VS Code
* Remote - SSH Extension
* Python Extension
* Jupyter Extension

Connect:

1. `Ctrl + Shift + P`
2. `Remote-SSH: Connect to Host`
3. Select:

```text
paramrudra
```

---

## 6. Verify Remote Connection

Open a terminal in VS Code:

```bash
hostname
pwd
```

Expected:

```text
login10
/home/IITB/CompnalAstrphyNRelvity/23n0315
```

---

## 7. FLASH Development

FLASH installation:

```text
/home/IITB/CompnalAstrphyNRelvity/23n0315/flash4.7.1
```

Open in VS Code:

```text
File → Open Folder
```

Select:

```text
flash4.7.1
```

---

## 8. Open Scratch Directory

Open scratch:

```text
File → Open Folder
```

Recommended:

```text
File → Add Folder to Workspace
```

Workspace structure:

```text
Workspace
├── flash4.7.1
└── scratch
```

This allows simultaneous access to:

* FLASH source code
* Simulation outputs
* Analysis scripts

---

## 9. Useful Param Rudra Commands

Login:

```bash
ssh paramrudra
```

Check jobs:

```bash
squeue -u 23n0315
```

Cancel a job:

```bash
scancel JOBID
```

View partitions:

```bash
sinfo
```

Check accounting:

```bash
sacct
```

---

## 10. Notes on tmux

Attempted:

```bash
tmux new -s flash
```

Result:

```text
tmux: command not found
```

Checked:

```bash
module avail tmux
module spider tmux
```

No tmux module available on Param Rudra.

Since VS Code Remote SSH reconnects automatically, work can continue without tmux.

---

## 11. Current Status

* [x] OpenSSH installed on Windows
* [x] SSH keys generated
* [x] Passwordless SSH login configured
* [x] SSH config file created
* [x] VS Code installed
* [x] Remote SSH configured
* [x] Connected successfully to Param Rudra
* [x] FLASH directory accessible
* [x] Ready for FLASH development

---

## Daily Workflow

```text
Windows Desktop
        │
        ▼
VS Code
        │
        ▼
Remote SSH
        │
        ▼
Param Rudra
        │
        ├── flash4.7.1
        ├── scratch
        └── SLURM Jobs
```

Typical session:

```bash
ssh paramrudra
```

Open VS Code → Connect to Param Rudra → Open FLASH workspace → Submit jobs → Analyze results.
