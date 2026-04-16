# Home Assistant Add-on: RustDesk Server

[![GitHub Release][releases-shield]][releases]
[![License][license-shield]](LICENSE)

![Supports amd64 Architecture][amd64-shield]
![Supports aarch64 Architecture][aarch64-shield]

Self-hosted [RustDesk][rustdesk] server (hbbs + hbbr) for your own
remote desktop relay, running as a Home Assistant add-on.

[![Open Home Assistant and add repository.][add-repo-shield]][add-repo]

## About

[RustDesk][rustdesk] is a full open-source remote desktop software, written in
Rust. It is an alternative to TeamViewer or AnyDesk. This add-on lets you
self-host the signaling (`hbbs`) and relay (`hbbr`) servers required by
RustDesk clients, so your remote desktop traffic stays on infrastructure you
control.

**No License Required:** Unlike the RustDesk Server PRO version, this add-on
bundles the free, open-source version of RustDesk Server. You do not need
any paid license or subscription to run this relay for your personal devices!

The public RustDesk servers are intended for testing and research; they are
not sized for production traffic. Self-hosting gives you:

- Predictable connection times and throughput.
- Full control over keys and who can connect to your relay.
- The option to enforce end-to-end encrypted clients only.

## Installation and Configuration

1. Click the button above to add this repository to your Home Assistant instance.
   *(Note: This only adds the repository to your Add-on Store, it doesn't
   install the addon directly).*
2. Navigate to Settings > Add-ons > Add-on Store.
3. Scroll all the way down to **RustDesk Server Home Assistant Add-on** and
   select it.
4. Click **Install**.
5. Start the add-on. (It auto-generates your `public_key` and `private_key`
   on the first boot).
6. Check the Add-on Logs for your generated keys and put them into your clients!

Detailed documentation lives in [DOCS.md](./rustdesk-server/DOCS.md).

### Using with an Internal Domain (NGINX)

If you want to use a domain name like `rustdesk.example.com`, note that
RustDesk uses raw TCP/UDP connections. A standard HTTP reverse proxy (like
a default Nginx web proxy) **will not work**.

You must configure **TCP/UDP Streams** for ports `21115`-`21117`. If you use
Nginx Proxy Manager, add these ports under the **Streams** tab, not the
Hosts tab! See [DOCS.md](./rustdesk-server/DOCS.md) for the exact
configuration snippet.

## Support

Got questions?

- Open an [issue on GitHub][issues].
- Check the [Home Assistant community forum][forum].
- Join the [Home Assistant Discord][discord-ha].

## License

MIT License - see [LICENSE](LICENSE) for details. Bundles the upstream
[RustDesk Server][rustdesk-server] binaries released by the
[RustDesk project][rustdesk].

[add-repo]: https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2Fsankeerthboddu%2Fha-rustdesk-server
[add-repo-shield]: https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg
[aarch64-shield]: https://img.shields.io/badge/aarch64-yes-green.svg
[amd64-shield]: https://img.shields.io/badge/amd64-yes-green.svg

[discord-ha]: https://discord.gg/c5DvZ4e
[forum]: https://community.home-assistant.io
[issues]: https://github.com/sankeerthboddu/ha-rustdesk-server/issues
[license-shield]: https://img.shields.io/github/license/sankeerthboddu/ha-rustdesk-server
[releases-shield]: https://img.shields.io/github/v/release/sankeerthboddu/ha-rustdesk-server
[releases]: https://github.com/sankeerthboddu/ha-rustdesk-server/releases
[rustdesk]: https://github.com/rustdesk/rustdesk
[rustdesk-server]: https://github.com/rustdesk/rustdesk-server
