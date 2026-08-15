# Home Assistant App: RustDesk Server

Self-hosted [RustDesk][rustdesk] server (`hbbs` + `hbbr`) for your own
remote-desktop relay, packaged as a Home Assistant App (formerly add-on).

## About

[RustDesk][rustdesk] is an open-source remote-desktop application. This App
bundles the two open-source RustDesk server components:

- `hbbs` - ID / rendezvous server.
- `hbbr` - relay server.

Hosting your own server gives you control over the server identity and relay
path and lets the service run alongside the rest of your Home Assistant
infrastructure.

## Installation

1. Add this repository to Home Assistant:

   [![Open Home Assistant and add repository.][add-repo-shield]][add-repo]

2. Open **Settings > Apps > App Store**. Older Home Assistant versions call
   this area **Add-ons** / **Add-on Store**.
3. Find **RustDesk Server** and select **Install**.
4. Leave the key fields empty unless you are restoring an existing keypair.
5. Start the App.
6. Recommended: enable **Start on boot** and **Watchdog**.
7. Open the App logs and copy the displayed public key into your RustDesk
   clients.

## Configuration

Example App configuration:

```yaml
encrypted_only: true
relay: ""
private_key: ""
public_key: ""
```

> [!TIP]
> **Configure both ends.** The controlling RustDesk client and the target
> RustDesk client must use the same private **ID Server** and public **Key**.
> A target still using RustDesk's public infrastructure can produce errors such
> as `ID Does Not Exist` even when the controlling client says `Ready`.

### Option: `encrypted_only`

When `true` (default), `hbbs` and `hbbr` start with `-k _`. Clients must use
the server key. This is strongly recommended for an Internet-reachable server.

### Option: `relay`

Optional hostname or IP address for the relay (`hbbr`) that `hbbs` advertises
to clients. In a normal single-host installation you can usually leave this
blank and let RustDesk clients infer the relay.

Set it when you deliberately need to advertise a different reachable address,
for example:

```yaml
relay: rustdesk.example.com
```

or a non-default relay port:

```yaml
relay: rustdesk.example.com:21117
```

Do not set `relay` merely because you are using Tailscale or a DNS name. First
verify that the address you advertise is actually reachable by both clients.

### Options: `private_key` and `public_key`

For a new installation, leave both blank. The App generates a keypair on first
boot and stores it persistently. The public key is printed in the App logs.

Only supply these values when you intentionally want to restore or reuse an
existing RustDesk server identity. Never post your private key in a GitHub
issue, forum post, or screenshot.

## Ports

The App declares the full RustDesk server port set:

| Port | Component | Purpose |
| --- | --- | --- |
| 21114/tcp | hbbs | Web console / API (primarily Pro features) |
| 21115/tcp | hbbs | NAT type test and online status |
| 21116/tcp | hbbs | TCP hole punching and connection service |
| 21116/udp | hbbs | ID registration and heartbeat |
| 21117/tcp | hbbr | Relay service |
| 21118/tcp | hbbs | Web-client support (optional) |
| 21119/tcp | hbbr | Web-client support (optional) |

For normal native-client access from the Internet, forward these to the Home
Assistant host:

- `21115/tcp`
- `21116/tcp`
- `21116/udp`
- `21117/tcp`

The App uses `host_network: true` so `hbbs` and `hbbr` bind on the Home
Assistant host network. Home Assistant still displays the declared ports in
the App UI.

## External access

RustDesk uses raw TCP and UDP sockets. A normal HTTP/HTTPS reverse proxy does
not proxy these connections.

### Direct router forward / DuckDNS

A simple public setup is:

1. Point your DNS name, such as `myhome.duckdns.org`, at your public IP.
2. Forward `21115/tcp`, `21116/tcp`, `21116/udp`, and `21117/tcp` to the Home
   Assistant host.
3. Configure that hostname as the **ID Server** on both RustDesk clients.
4. Configure the public key from the App logs on both clients.

### Cloudflare-managed DNS

If Cloudflare manages the domain, create a DNS record that points to your
public IP and leave the record **DNS Only** (grey cloud).

> [!WARNING]
> The normal Cloudflare HTTP proxy does not carry RustDesk's raw TCP/UDP
> protocol. An orange-cloud proxy record is not a substitute for forwarding
> RustDesk's ports.

### Nginx Proxy Manager / NGINX Streams

Do not use an ordinary HTTP **Proxy Host** for RustDesk. Use TCP/UDP Streams.
For Nginx Proxy Manager, create these streams to the Home Assistant host:

- `21115/tcp` -> `<HA_IP>:21115`
- `21116/tcp` -> `<HA_IP>:21116`
- `21116/udp` -> `<HA_IP>:21116`
- `21117/tcp` -> `<HA_IP>:21117`

A manual NGINX example:

```nginx
stream {
    server {
        listen 21115;
        proxy_pass <YOUR_HA_IP>:21115;
    }

    server {
        listen 21116;
        proxy_pass <YOUR_HA_IP>:21116;
    }

    server {
        listen 21116 udp;
        proxy_pass <YOUR_HA_IP>:21116;
    }

    server {
        listen 21117;
        proxy_pass <YOUR_HA_IP>:21117;
    }
}
```

## Tailscale

Tailscale can be a good way to keep RustDesk private, but there are two common
network layouts and they behave differently on Home Assistant.

### Recommended: Home Assistant LAN IP through a subnet route

This is usually the easiest layout to reason about:

1. Advertise the Home Assistant LAN subnet from an appropriate Tailscale node
   and approve the route in Tailscale.
2. From the remote Tailscale client, verify that the Home Assistant LAN IP is
   reachable.
3. Configure the Home Assistant LAN IP (for example `192.168.1.20`) as the
   RustDesk **ID Server** on both RustDesk clients.
4. Leave `relay` blank initially.
5. Use the public key from the App logs on both clients.

Traffic remains reachable over Tailscale, while RustDesk talks to the same LAN
address on which the Home Assistant host-networked App is already listening.

### Direct Home Assistant Tailscale `100.x` address

This can work, but it depends on how the Home Assistant Tailscale App exposes
its network interface. Tailscale commonly supports userspace networking, where
having a `100.x` address does not necessarily mean every host-networked App is
bound directly to that address in the way a normal Linux `tailscale0`
interface would be.

If clients show **Ready** using the `100.x` address but an actual session fails
with **connection reset by peer**, do not assume the RustDesk server itself is
broken. Test the LAN-IP-over-subnet-route layout above. Also verify that you
have not set `relay` to an address that one of the two clients cannot reach.

If you deliberately disable Tailscale userspace networking and expose a normal
host `tailscale0` interface, direct `100.x` addressing may be appropriate, but
that is a Home Assistant/Tailscale networking decision rather than a setting
this RustDesk App controls.

### What `Ready` proves

A green **Ready** state is useful but is not an end-to-end test of a remote
session. It means the client can perform enough rendezvous communication to
consider the configured server available. A successful remote session can
still require a working hole-punch or relay path between both clients.

When troubleshooting Tailscale, collect these four facts separately:

1. Can **both** RustDesk clients reach the configured ID Server address?
2. Do **both** clients use the same public key?
3. Is `relay` blank, or does it advertise an address reachable by both clients?
4. Does the session work using the Home Assistant LAN IP over a Tailscale
   subnet route?

## Troubleshooting

### `ID Does Not Exist`

Usually one client is still registered against a different RustDesk server.
Verify the **ID Server** and **Key** on both the controlling and target clients.

### `connection reset by peer`

This normally means the initial server contact was not the whole problem. Check
which address is being advertised for the relay, whether both clients can reach
it, and whether the same setup works using a LAN address. For Tailscale, see
the dedicated section above.

### Client says `Ready`, but a session still fails

Check the actual transport path rather than only the status indicator:

- Verify both clients use the same ID Server and public key.
- Temporarily leave `relay` blank.
- If using the public Internet, confirm the required TCP/UDP forwarding.
- If using NGINX/NPM, confirm you used Streams rather than HTTP Proxy Hosts.
- If using Tailscale, test the LAN address through a subnet route.

### Cloudflare log viewer timeout

If you access Home Assistant itself through an HTTP proxy such as Cloudflare,
the Home Assistant log-stream web page can time out while the quiet RustDesk
App continues running. Refresh the Home Assistant page and check the App state
before treating a browser gateway error as a RustDesk service crash.

## Persistent data and backup

The App persists its keys and `hbbs` SQLite data under its Home Assistant App
configuration directory. Include App configuration data in your Home Assistant
backup strategy if retaining the same RustDesk server identity matters to you.

Do not casually regenerate the keypair after clients are deployed; doing so
requires updating the public key on those clients.

## Upgrading

The Dockerfile pins the upstream RustDesk Server version and SHA-256 digests.
Renovate can propose a new RustDesk version, but CI intentionally requires the
upstream asset digests to be updated as part of that review.

The Home Assistant App version adds a packaging revision to the upstream
RustDesk version. For example:

- `1.1.16-1` = RustDesk Server `1.1.16`, packaging revision `1`.
- `1.1.16-2` = the same RustDesk Server version with App packaging changes.

## Support

- [Open a GitHub issue][issues] for reproducible App bugs.
- Use the [Home Assistant community thread][forum] for setup and networking
  discussion.
- Read [SECURITY.md][security] before reporting a sensitive vulnerability.

[add-repo]: https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2Fsankeerthboddu%2Fha-rustdesk-server
[add-repo-shield]: https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg
[forum]: https://community.home-assistant.io/t/self-host-rustdesk-server-inside-home-assistant/1004868
[issues]: https://github.com/sankeerthboddu/ha-rustdesk-server/issues
[rustdesk]: https://github.com/rustdesk/rustdesk
[security]: https://github.com/sankeerthboddu/ha-rustdesk-server/blob/main/SECURITY.md
