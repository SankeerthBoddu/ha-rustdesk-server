# Home Assistant App: RustDesk Server

[![GitHub Release][releases-shield]][releases]
[![License][license-shield]](LICENSE)

![Supports amd64 Architecture][amd64-shield]
![Supports aarch64 Architecture][aarch64-shield]

Self-hosted [RustDesk][rustdesk] server (`hbbs` + `hbbr`) for your own
remote-desktop relay, packaged as a Home Assistant App (formerly add-on).

[![Open Home Assistant and add repository.][add-repo-shield]][add-repo]

## About

[RustDesk][rustdesk] is open-source remote-desktop software and an alternative
to services such as TeamViewer or AnyDesk. This App lets you self-host the
signaling (`hbbs`) and relay (`hbbr`) services required by RustDesk clients,
so the server side of your remote-access setup stays on infrastructure you
control.

**No paid RustDesk Server license is required.** This App bundles the free,
open-source RustDesk Server, not RustDesk Server Pro.

Self-hosting gives you:

- Predictable connection times and throughput.
- Control over your server identity and public key.
- The option to require clients to use the server key.
- A RustDesk server that can live alongside the rest of your Home Assistant
  infrastructure.

## Installation and configuration

1. Click the button above to add this repository to Home Assistant.
2. Open **Settings > Apps > App Store** (called **Add-ons** on older Home
   Assistant versions).
3. Find **RustDesk Server** and select **Install**.
4. Start the App. It generates a keypair on first boot if you leave the key
   fields empty.
5. Open the App logs and copy the displayed public key into your RustDesk
   clients.
6. Configure the same ID Server and public key on every client that should use
   this private server.

Detailed configuration, networking, Tailscale, Cloudflare, and troubleshooting
guidance lives in [DOCS.md](./rustdesk-server/DOCS.md).

## Networking in one minute

RustDesk uses raw TCP and UDP traffic, not ordinary HTTP. For direct Internet
access, the normally required ports are:

- `21115/tcp`
- `21116/tcp`
- `21116/udp`
- `21117/tcp`

A normal HTTP reverse-proxy host will not carry this traffic. If you use Nginx
Proxy Manager, use **Streams** rather than a regular Proxy Host. If Cloudflare
manages the hostname, the RustDesk DNS record must not be proxied through the
normal Cloudflare HTTP proxy.

For Tailscale, do not assume that a Home Assistant host-networked App can
always use the host's `100.x` Tailscale address directly. If direct Tailscale
addressing gives `connection reset by peer`, using the Home Assistant LAN IP
through an approved Tailscale subnet route is often the simpler layout. See
[DOCS.md](./rustdesk-server/DOCS.md) for both patterns and checks.

## Maintenance

This project is actively maintained as a small community App:

- Renovate monitors RustDesk, the Home Assistant base image, and GitHub Actions.
- CI lints the App and builds both `amd64` and `aarch64` images before merge.
- Upstream RustDesk archives are verified against pinned SHA-256 digests.
- CI blocks fixable high/critical OS-package vulnerabilities and reports
  inherited library findings from the Home Assistant base image.
- Security-impacting RustDesk releases are prioritized, but this project does
  not promise a formal support SLA.

The bundled RustDesk version and the Home Assistant packaging revision are
tracked separately. For example, App version `1.1.16-2` means RustDesk Server
`1.1.16`, packaging revision `2`.

## Support

For setup and networking questions, the [Home Assistant community thread][forum]
is usually the best place to compare working configurations. For reproducible
App bugs, open a [GitHub issue][issues] and include the requested environment
and networking details.

For sensitive security reports, see [SECURITY.md](SECURITY.md).

## License

MIT License - see [LICENSE](LICENSE) for details. The App downloads upstream
[RustDesk Server][rustdesk-server] binaries released by the
[RustDesk project][rustdesk].

[add-repo]: https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2Fsankeerthboddu%2Fha-rustdesk-server
[add-repo-shield]: https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg
[aarch64-shield]: https://img.shields.io/badge/aarch64-yes-green.svg
[amd64-shield]: https://img.shields.io/badge/amd64-yes-green.svg
[forum]: https://community.home-assistant.io/t/self-host-rustdesk-server-inside-home-assistant/1004868
[issues]: https://github.com/sankeerthboddu/ha-rustdesk-server/issues
[license-shield]: https://img.shields.io/github/license/sankeerthboddu/ha-rustdesk-server
[releases-shield]: https://img.shields.io/github/v/release/sankeerthboddu/ha-rustdesk-server
[releases]: https://github.com/sankeerthboddu/ha-rustdesk-server/releases
[rustdesk]: https://github.com/rustdesk/rustdesk
[rustdesk-server]: https://github.com/rustdesk/rustdesk-server
