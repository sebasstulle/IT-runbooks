# SSH key-based authentication setup

**Environment:** Debian 13· VirtualBox (NAT) · Tabby on Windows  
**Date:** 2026-04

---

## Symptom

Tabby was rejecting the connection to the Debian VM.
Not clear at first whether the issue was the network,
the SSH service, or the credentials.

---

## Diagnosis

**Step 1 — Check if SSH is running**

First thing I verified was the SSH service itself:

    systemctl status ssh

Output showed `active (running)` and `enabled` — so SSH wasn't the problem.

**Step 2 — Check the network**

The VM is on NAT, so it's not directly reachable from Windows.
Configured port forwarding in VirtualBox:

    Settings → Network → Adapter 1 → Advanced → Port Forwarding
    
    Name:       SSH
    Protocol:   TCP
    Host IP:    127.0.0.1
    Host Port:  2222
    Guest IP:   (empty)
    Guest Port: 22

**Step 3 — Found the real problem**

Authentication was failing because the username was typed with a capital S in Tabby.
Linux usernames are case-sensitive — `Sysadm` and `sysadm` are different users.

Lesson: always verify credentials exactly, including case.

---

## Setting up key-based authentication

Once connected with password, set up SSH keys to avoid typing a password on every login.

**Step 1 — Generate the key pair on Debian**

    ssh-keygen -t ed25519 -C "lab-debian"

This generates two files:

    ~/.ssh/id_ed25519      → private key (never share this)
    ~/.ssh/id_ed25519.pub  → public key (this goes on the server)

The idea: the public key is a lock placed on the server.
The private key is the only key that opens it.
Without both parts, no one gets in.

**Step 2 — Add public key to authorized_keys**

    nano ~/.ssh/authorized_keys
    # paste the contents of id_ed25519.pub
    # Ctrl+O to save, Ctrl+X to exit

**Step 3 — Set correct permissions**

SSH is strict about this. If permissions are wrong, it refuses to use the keys at all.

    chmod 700 ~/.ssh
    chmod 600 ~/.ssh/authorized_keys
    chmod 600 ~/.ssh/id_ed25519
    chmod 644 ~/.ssh/id_ed25519.pub

What these mean:

    700 → only owner can read, write, and enter the directory
    600 → only owner can read and write
    644 → owner can read/write, everyone else can only read

**Step 4 — Configure Tabby to use the key**

In Tabby, edited the profile and changed Authentication
from Password to Key, pointing it to the private key file.

---

## Result

Tabby connects to Debian without prompting for a password.
Authentication uses the private key automatically.

---

## What I learned

- Linux usernames are case-sensitive — always double-check
- SSH won't work if file permissions are incorrect
- The private key never travels over the network — that's what makes key auth more secure than passwords
- Port forwarding is required when the VM is on NAT

---

*Tested on: Debian 13 · VirtualBox 7 · Tabby terminal*
