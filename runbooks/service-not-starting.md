# Service not starting after reboot

Environment: Debian 13 · VirtualBox 7 · Apache2  
Date: 2026-05

## Symptom

Rebooted the VM and tried to open the lab website in the browser.
Got a "connection refused" error instead of the page.

Wasn't sure if the problem was the network, the service, or something
that broke during the reboot.

## Diagnosis

### Step 1 — Check if the service is running

First thing I checked was the service status:

```bash
systemctl status apache2
```

Output showed the service was inactive (dead) — it wasn't running
and hadn't started automatically after the reboot.

### Step 2 — Check the logs

To understand why it failed, I looked at the journal logs:

```bash
journalctl -u apache2 --no-pager | tail -20
```

The error was clear: port 80 was already in use by another process.

### Step 3 — Find what was using port 80

```bash
ss -tlnp | grep :80
```

Found that a Python simple HTTP server I had started manually the day
before was still running and holding port 80.

## Resolution

### Step 1 — Kill the process using port 80

Got the PID from the previous ss command and killed it:

```bash
kill 1234
```

Verified the port was free:

```bash
ss -tlnp | grep :80
```

No output — port was free.

### Step 2 — Start the service

```bash
systemctl start apache2
```

### Step 3 — Verify it's running

```bash
systemctl status apache2
```

Output showed active (running). Opened the browser and the lab
website loaded correctly.

### Step 4 — Make sure it starts automatically on reboot

```bash
systemctl enable apache2
```

This was the real fix — the service wasn't enabled to start
automatically. That's why it died after the reboot.

## What I learned

- `systemctl status` is always the first command when a service isn't working
- A service can be running but not enabled — it won't survive a reboot
- `journalctl -u servicename` shows exactly why a service failed
- `ss -tlnp` is useful to find what's occupying a port
- The difference between `start` and `enable`: start runs it now,
  enable makes it run on every boot. You usually want both.
