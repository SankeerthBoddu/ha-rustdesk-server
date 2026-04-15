# Home Assistant Add-on: RustDesk Server

Self-hosted [RustDesk][rustdesk] server (`hbbs` + `hbbr`) for your own
remote-desktop relay.

## About

[RustDesk][rustdesk] is an open-source remote-desktop application written in
Rust. The public RustDesk servers are meant for testing and are not sized for
production traffic. Hosting your own server gives you:

- Predictable connection times and throughput.
- Control over which keys are allowed to relay traffic.
- The option to enforce end-to-end encrypted clients only.

This add-on bundles the two RustDesk server components:

- `hbbs` - the ID / rendezvous server.
- `hbbr` - the relay server.

## Installation

1. Add this repository to your Home Assistant add-on store:

   [![Open your Home Assistant instance and show the add add-on repository dialog with a specific repository URL pre-filled.][add-repo-shield]][add-repo]

2. Search for **RustDesk Server** in the add-on store and click **Install**.
3. Configure the add-on (see below) and start it.
4. (Recommended) Enable **Watchdog** and **Start on boot**.

## Configuration

Example add-on configuration:

```yaml
encrypted_only: true
relay: rustdesk.example.com
private_key: ""
public_key: ""
```

> [!NOTE]
> Leaving `private_key` / `public_key` empty lets `hbbs` generate a fresh
> keypair on first boot. The generated keys are stored in the add-on
> configuration folder (`/addon_configs/<slug>/`) as `id_ed25519` and
> `id_ed25519.pub`. Copy the contents of `id_ed25519.pub` into your RustDesk
> clients as the **Key**.

### Option: `encrypted_only`

When `true` (default), `hbbs` and `hbbr` are started with `-k _`, which forces
all clients to use the server's key and rejects unencrypted clients. Strongly
recommended for any public deployment.

### Option: `relay` (optional)

Hostname or IP address of the machine running `hbbr` (usually the same host as
this add-on). Clients will be told to use this address for relayed
connections. Include a port only if you use something other than the default
`21117`, for example `rustdesk.example.com:21117`. Separate multiple relays
with commas.

### Option: `private_key` (optional)

The Ed25519 private key (base64) for the server. Leave empty to auto-generate
on first boot.

### Option: `public_key` (optional)

The Ed25519 public key (base64) matching `private_key`. Must be provided
together with `private_key` if you supply one.

## Ports

The add-on exposes the full set of ports RustDesk uses. Forward these on your
router to reach the server from the outside:

| Port        | Component | Purpose                                                       |
| ----------- | --------- | ------------------------------------------------------------- |
| 21114/tcp   | hbbs      | Web console / API (Pro feature, not needed for OSS clients)   |
| 21115/tcp   | hbbs      | NAT type test and online status query (**required**)          |
| 21116/tcp   | hbbs      | TCP hole punching and connection service (**required**)       |
| 21116/udp   | hbbs      | ID registration and heartbeat (**required**)                  |
| 21117/tcp   | hbbr      | Relay service (**required**)                                  |
| 21118/tcp   | hbbs      | RustDesk Web Client support (optional)                        |
| 21119/tcp   | hbbr      | RustDesk Web Client support (optional)                        |

> [!TIP]
> The add-on runs on the host network to allow accurate NAT type detection
> and UDP hole punching. Ports are still declared so the Supervisor shows
> them in the UI.

## Reverse Proxy / NGINX

RustDesk relies heavily on raw TCP and UDP connections. Standard HTTP/HTTPS reverse proxies (like typical Nginx setups for web interfaces) **will not work** out-of-the-box for RustDesk's core components.

If you are using NGINX or Nginx Proxy Manager to route traffic via an internal domain like `rustdesk.your-domain.tld`, you must configure TCP/UDP streams.

### Nginx Proxy Manager (NPM Add-on)
If you use the Nginx Proxy Manager Add-on in Home Assistant:
1. Go to **Streams** (not Hosts).
2. Add four separate streams targeting your Home Assistant IP address:
   - Port `21115` TCP -> `<HA_IP>:21115`
   - Port `21116` TCP -> `<HA_IP>:21116`
   - Port `21116` UDP -> `<HA_IP>:21116`
   - Port `21117` TCP -> `<HA_IP>:21117`
3. Ensure these ports are open and forwarded on your edge router to the NPM host.

### NGINX Config (Manual)
Add the following to your `nginx.conf` within the `stream { ... }` block (outside the `http { ... }` block):

```nginx
stream {
    # hbbs - NAT type test and online status
    server {
        listen 21115;
        proxy_pass <YOUR_HA_IP>:21115;
    }
    # hbbs - Hole punching
    server {
        listen 21116;
        proxy_pass <YOUR_HA_IP>:21116;
    }
    # hbbs - UDP heartbeat
    server {
        listen 21116 udp;
        proxy_pass <YOUR_HA_IP>:21116;
    }
    # hbbr - Relay service
    server {
        listen 21117;
        proxy_pass <YOUR_HA_IP>:21117;
    }
}
```

## Persistent data

The add-on persists its keys and the `hbbs` SQLite database under
`/addon_configs/<slug>/` on your Home Assistant host. Back this directory up
if you want to restore the same identity after a reinstall.

## Upgrading

The pinned `rustdesk-server` version is shown in the top of the add-on
`Dockerfile`. When upgrading, make sure your clients are compatible with the
new server version - RustDesk occasionally changes the wire protocol.

## Support

- [Open an issue on GitHub][issues]
- [Home Assistant community forum][forum]
- [Home Assistant Discord][discord-ha]

[add-repo]: https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2Fsankeerthboddu%2Fha-rustdesk-server
[add-repo-shield]: https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg
[discord-ha]: https://discord.gg/c5DvZ4e
[forum]: https://community.home-assistant.io
[issues]: https://github.com/sankeerthboddu/ha-rustdesk-server/issues
[rustdesk]: https://github.com/rustdesk/rustdesk
