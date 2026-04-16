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

   [![Open Home Assistant and add repository.][add-repo-shield]][add-repo]

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

> [!TIP]
> **Connecting Clients**
> To successfully establish a RustDesk remote session, BOTH your local
> machine and your target remote machine MUST be configured identically. Open
> the **Network Settings** in both RustDesk apps and specify your **ID Server**
> (your domain or IP) and paste your **Key** (the public key shown in logs).
> When correctly configured, the bottom of the client will turn green and say
> "Ready".

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

### Option: `private_key` & `public_key` (Leave Blank)

There is **no need** to manually fill out the `private_key` and `public_key`
fields in the add-on config. Simply leave them empty!

When the add-on boots for the first time, it automatically generates a fresh,
secure keypair for you. You can easily find the `Key` printed directly in the
Add-on's **Log** tab inside Home Assistant. Copy that key and paste it into
your RustDesk clients.

## Ports

The add-on exposes the full set of ports RustDesk uses. Forward these on your
router to reach the server from the outside:

| Port | Component | Purpose |
| --- | --- | --- |
| 21114/tcp | hbbs | Web console / API (Pro only) |
| 21115/tcp | hbbs | NAT type test and online status (**required**) |
| 21116/tcp | hbbs | TCP hole punching (**required**) |
| 21116/udp | hbbs | ID registration and heartbeat (**required**) |
| 21117/tcp | hbbr | Relay service (**required**) |
| 21118/tcp | hbbs | RustDesk Web Client support (optional) |
| 21119/tcp | hbbr | RustDesk Web Client support (optional) |

> [!TIP]
> The add-on runs on the host network to allow accurate NAT type detection
> and UDP hole punching. Ports are still declared so the Supervisor shows
> them in the UI.

## External Access & Domains

Because RustDesk uses custom TCP and UDP sockets for hole-punching, your
networking setup must be configured carefully. Standard website reverse-proxies
will not work out-of-the-box.

### Option 1: Direct Router Forward (DuckDNS / No Proxy)

If you use **DuckDNS** or just want a simple setup, you do not need NGINX at
all! Your domain (e.g., `myhome.duckdns.org`) resolves directly to your home
IP without HTTP proxying.

1. Log into your home router's settings.
2. Port Forward the following ports to your Home Assistant IP address:
   - `21115` (TCP)
   - `21116` (TCP and UDP)
   - `21117` (TCP)
3. In your RustDesk Client, enter `myhome.duckdns.org` as your **ID Server**.

### Option 2: Cloudflare (Custom Domain)

If you own a custom domain (e.g., `rustdesk.your-domain.com`) managed by
Cloudflare, you must configure your DNS `A` record as **DNS Only (Grey Cloud)**.

> [!WARNING]
> If you leave the Cloudflare Proxy enabled (Orange Cloud), Cloudflare will
> instantly block all of RustDesk's traffic because it is not standard website
> HTTP/HTTPS traffic.

1. Create an `A` record in Cloudflare pointing to your home IP address.
2. Click the Orange Cloud to turn it Grey (**DNS Only**).
3. Port Forward `21115-21117` on your router to your Home Assistant IP.

### Option 3: Advanced NGINX Streams

If you absolutely must run the traffic through an internal NGINX server (e.g.,
Nginx Proxy Manager), you cannot use standard HTTP "Proxy Hosts". You must
configure raw TCP/UDP Streams.

#### Nginx Proxy Manager (NPM Add-on)

1. Delete any "Proxy Hosts" you created for RustDesk.
2. Go to the **Streams** tab.
3. Add four separate streams targeting your Home Assistant IP address:
   - Port `21115` TCP -> `<HA_IP>:21115`
   - Port `21116` TCP -> `<HA_IP>:21116`
   - Port `21116` UDP -> `<HA_IP>:21116`
   - Port `21117` TCP -> `<HA_IP>:21117`
4. Port Forward these ports on your router, pointing them to your NPM host.

#### NGINX Config (Manual)

Add the following to your `nginx.conf` within the `stream { ... }` block
(outside the `http { ... }` block):

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

## Common Pitfalls & Troubleshooting

### 1. Error: "ID Does Not Exist"

If you set up your client and get an **"ID Does Not Exist"** error when trying
to remote into a machine, it means your local computer connects to your
Private Server correctly, but the *Target Computer* does not! Every computer
you wish to control must have their **ID Server** and **Key** configured to
point to your Private Server.

### 2. Cloudflare 504 Gateway Timeout (Checking Logs)

If you are viewing your Home Assistant Add-on logs remotely via a Cloudflare
proxy, your log viewer tab might suddenly crash with an HTML
**504 Gateway Timeout** error.

- **Why:** Cloudflare kills any HTTP connection that remains idle for 100
  seconds. Because the RustDesk server is quiet and rarely prints logs,
  Cloudflare assumes the log stream froze and cuts you off.
- **Fix:** Just refresh the web page! Your server did not crash.

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
