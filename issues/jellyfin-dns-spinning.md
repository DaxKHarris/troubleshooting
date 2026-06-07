# Jellyfin DNS Spinning Indefinitely

## Symptoms

- Accessing Jellyfin via DuckDNS domain spins indefinitely in the browser
- No error message — just hangs forever
- Affects some machines but not others
- Local IP access works fine on all machines
- External machines also affected intermittently

## What I Ruled Out

- **DNS resolution** — `dig yourname.duckdns.org` returned the correct public IP. DNS was working fine.
- **IP mismatch** — DuckDNS was up to date and pointing at the correct public IP.
- **NAT hairpinning** — Initially suspected since local IP worked but domain didn't. Ruled out because external machines were also affected.
- **ISP blocking / port forwarding flakiness** — Ruled out because local IP access never failed during the same period.
- **`search .` missing from `/etc/resolv.conf`** — Found on one machine but irrelevant since the full domain was being used.

## Root Cause

The server's firewall only allowed **HTTPS** traffic, but the browser was requesting over **HTTP**. Port 80, for HTTP, was not opened or forwarded.

This causes the browser to silently hang rather than immediately error out, because:
- DNS resolves correctly ✓
- The request leaves the client ✓
- It hits a closed port on the host and gets no response ✗
- The browser waits indefinitely for a reply that never comes

The reason it worked on some machines is that those browsers had HTTPS either cached, or automatically when they used their browser's autofill.

## Fix

Two valid options:

**Option A — Force HTTPS everywhere (What I did)**
Make sure all clients are accessing via `https://` and redirect HTTP to HTTPS on the server.

**Option B — Open HTTP port**
Forward port 80 on your router to the Jellyfin server if you intentionally want to serve over HTTP.

## Choice

I chose Option A because allowing HTTP over the world wide web is just asking for a man in the middle to gain access to the credentials that are in plain text, and either having valuable credentials or a session cookie.
