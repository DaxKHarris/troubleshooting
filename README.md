# Troubleshooting

A collection of documented issues I've encountered and resolved, including symptoms, what was ruled out, and the actual root cause.

## Issues

| # | Issue | Project | Summary | Link |
|---|-------|---------|---------|------|
| 1 | Jellyfin DNS spinning indefinitely | [Jellyfin Home Media Server](https://github.com/daxkharris/home-projects/blob/main/projects/jellyfin-server.md) | Browser silently failing due to HTTP port never being opened on the server | [details](issues/jellyfin-dns-spinning.md) |
| 2 | Server hanging on boot due to missing drive | [Jellyfin Home Media Server](https://github.com/daxkharris/home-projects/blob/main/projects/jellyfin-server.md) | Debian installer left a stale fstab entry for a removed drive, causing systemd to hang indefinitely on boot | [details](issues/fstab-missing-drive-hang.md) |

| 3 | Jellyfin server waking unexpectedly | [Jellyfin Home Media Server](https://github.com/daxkharris/home-projects/blob/main/projects/jellyfin-server.md) | Ghost process from decommissioned wake script firing WoL on every port scanner probe via CT-log-exposed domain | [details](issues/jellyfin-phantom-wakes.md) |
