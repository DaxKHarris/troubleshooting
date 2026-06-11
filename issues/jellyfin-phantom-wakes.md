# Jellyfin Server Waking Unexpectedly

## Symptoms

* Jellyfin server suspending and waking repeatedly on its own
* No active users during wake events
* Server would wake, idle for 20 minutes, suspend, then wake again almost immediately
* 79 wake events logged over 3 days
* Inactivity script working correctly — idle timer was functioning as expected

## What I Ruled Out

* **WoL firing on a running machine** — A WoL magic packet sent to an already-running machine is ignored at the hardware level. Not the cause.
* **Caddy health checks** — Removed `health_uri` and `health_interval` from the Caddyfile when switching to the exec-based wake approach. Not the cause.
* **Internet scanners triggering exec** — Raw TCP SYN probes from internet scanners complete a TCP handshake with Caddy but never send a valid HTTPS request, so Caddy's exec block never fires. Not the cause.

## Root Cause

The old `wake-server.sh` script was still running as a live process despite its systemd service being disabled and the script file being deleted from disk.

The script used `tcpdump` to monitor port 443 and fire a WoL magic packet at any TCP traffic — including raw port scanner SYN probes from the public internet. Disabling the systemd service only prevents future starts; it does not kill an already-running process. Deleting the file from disk also has no effect once a process is executing from memory.

Certificate transparency logs exposed the DuckDNS domain publicly when Let's Encrypt issued a TLS certificate. Bots scrape CT logs continuously, meaning internet scanners found the domain and probed port 443 regularly — each probe triggering a WoL packet from the still-running script.

## How I Found It

Correlated ping output against tcpdump timestamps to confirm a specific external IP woke the server:

```
[17:47:19] 100% packet loss        ← server asleep
[17:47:25] XXX.XX.XX.XXX hits :443 ← inbound connection attempt
[17:47:37] 64 bytes from 192.168.0.8 ← server awake
```

`whois XXX.XX.XX.XXX` identified the IP as a Norwegian hosting provider — a known scanner/bot host, not a legitimate user.

`ps aux | grep wake` revealed the ghost process:

```
533  Jun03  /bin/bash /home/daxkharris/Documents/WOL/wake-server.sh
```

Running since June 3rd, surviving service disable, file deletion, and multiple Caddy reloads.

## Fix

Killed the ghost process:

```bash
sudo kill 533
```

Rebooted the Pi to ensure a clean process tree:

```bash
sudo reboot
```

Confirmed only the correct wake-trigger was running afterward:

```bash
ps aux | grep wake
```

## Lesson

Disabling a systemd service does not kill its already-running process. Always kill the PID explicitly or reboot when decommissioning a long-running script. Verify with `ps aux` after, not before.
