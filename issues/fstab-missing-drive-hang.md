# Server Hanging on Boot Due to Missing Drive in fstab

## Symptoms

- Server appeared to be on — fans spinning, lights on
- No network response, could not be pinged
- No SSH access
- Required physically connecting a monitor to diagnose

## Background

A 500GB SSD was temporarily installed and mounted on the server. When it was swapped out for a 4TB HDD, the fstab entry written by the Debian installer was left behind. It referenced the old drive by UUID, which no longer existed.

## Root Cause

The Debian installer automatically wrote an fstab entry for the 500GB SSD when it was mounted. When the drive was removed, the entry remained, pointing at a UUID that no longer existed on the system.

On boot, systemd attempted to mount the drive, got no response, and hung indefinitely waiting for a drive that would never appear. There was no timeout, and no `nofail` flag to tell the system to continue booting regardless.

This caused the server to silently stall — powered on, but never finishing the boot process, leaving it unreachable over the network.

## Diagnosis

Connecting a monitor revealed the server had dropped into **emergency mode** — a recovery root shell systemd falls back to when a critical mount fails. The error indicated a drive could not be found.

## Fix

From emergency mode, edited fstab to remove the stale entry and added `nofail` to any non-critical drives:

```bash
nano /etc/fstab
```

The `nofail` option tells systemd to continue booting even if the drive is not found, rather than hanging:

```
UUID=xxxx-xxxx  /mnt/storage  ext4  defaults,nofail  0  2
```

This ensures that if a drive is removed or dies, the server stays online instead of becoming unreachable.
