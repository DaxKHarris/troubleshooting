# Troubleshooting

A collection of documented issues I've encountered and resolved, including symptoms, what was ruled out, and the actual root cause.

## Issues

| # | Issue | Summary | Link |
|---|-------|---------|------|
| 1 | Jellyfin DNS spinning indefinitely | Browser silently failing due to HTTP port never being opened on the server | [details](issues/jellyfin-dns-spinning.md) |
| 2 | Server hanging on boot due to missing drive | Debian installer left a stale fstab entry for a removed drive, causing systemd to hang indefinitely on boot | [details](issues/fstab-missing-drive-hang.md) |
