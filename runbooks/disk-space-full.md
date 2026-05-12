# Disk space full

Environment: Debian 13 · VirtualBox 7  
Date: 2026-05

## Symptom

The system started behaving strangely — commands were slow,
services were throwing errors, and one of them failed to write a log file.
Suspected the disk was full but wasn't sure.

## Diagnosis

### Step 1 — Check disk usage

```bash
df -h
```

Output showed one partition at 100% usage.

### Step 2 — Find what was taking the space

Navigated to the full partition and ran:

```bash
du -sh * | sort -rh | head -10
```

This sorts everything by size, biggest first.
Found a folder with old log files taking several gigabytes.

### Step 3 — Confirm before deleting

```bash
ls -lh /var/log/
```

Old compressed logs were accumulating — some hadn't been
rotated properly.

## Resolution

### Step 1 — Remove old logs

```bash
rm /var/log/apache2/*.gz
```

Only removed compressed rotated logs — never touch active log files.

### Step 2 — Verify space was recovered

```bash
df -h
```

Partition back to normal usage.

### Step 3 — Check logrotate is configured

```bash
cat /etc/logrotate.d/apache2
```

Confirmed logrotate was set up to rotate logs automatically going forward.

## What I learned

- `df -h` shows disk usage per partition — first command when suspecting full disk
- `du -sh * | sort -rh` finds what's eating the space
- Never delete active log files — only old rotated ones (.gz files are safe)
- Services fail in unexpected ways when disk is full — it's not always obvious
- logrotate prevents this from happening automatically if configured correctly
