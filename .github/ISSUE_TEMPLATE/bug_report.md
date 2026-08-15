---
name: Bug report
about: Report a reproducible problem with the Home Assistant App
labels: "bug"
assignees: ""
---

# Bug report

## Problem

Describe what happens and what you expected instead.

## Steps to reproduce

1. ...
2. ...
3. ...

## Environment

- Home Assistant App version:
- Home Assistant OS / installation type:
- Home Assistant Core version:
- Supervisor version, if applicable:
- Architecture: `amd64` / `aarch64`
- RustDesk client version on controlling device:
- RustDesk client version on target device:

## Network path

- Connection type: LAN / public Internet / Tailscale / other
- ID Server configured on both clients:
  - You may redact a private hostname or IP, but say whether both clients use
    the same value.
- App `relay` option: blank / configured (provide the value if safe)
- Do both RustDesk clients show `Ready`? yes / no
- If using public Internet, are these forwarded to the Home Assistant host?
  - `21115/tcp`
  - `21116/tcp`
  - `21116/udp`
  - `21117/tcp`
- Are you using Cloudflare, Nginx Proxy Manager / NGINX Streams, Tailscale,
  CGNAT, or another proxy/VPN? Describe briefly.
- If using Tailscale, did you also test the Home Assistant LAN IP through a
  Tailscale subnet route? What happened?

## Logs

Paste the relevant Home Assistant App logs from startup through the failure.

```text
logs here
```

> [!WARNING]
> Do not post your RustDesk private key, Home Assistant tokens, Tailscale auth
> keys, passwords, or other secrets. The RustDesk **public** key is not secret,
> but it normally is not needed for a bug report.

## Additional context

Add screenshots, topology details, or anything else that may help reproduce the
problem. Redact public IPs, private domains, or device names if you prefer.
