# Next Steps

Everything in this repo is already prepared for the new GitHub repo
`sankeerthboddu/ha-rustdesk-server`. All file contents (URLs, image paths,
labels, docs) point there.

Follow the steps below **in order**.

---

## Step 1 - Push this code into `ha-rustdesk-server`

Pick ONE option. Option A gives you a clean single-commit history. Option B
keeps every commit from this repo.

### Option A - Clean slate (recommended)

From a terminal on your machine:

```bash
# 1. Clone this repo locally (if you have not already)
git clone https://github.com/SankeerthBoddu/addon-rustdesk-server.git
cd addon-rustdesk-server

# 2. Make sure you have the latest main
git checkout main
git pull origin main

# 3. Build a fresh copy without .git
cd ..
mkdir ha-rustdesk-server
rsync -a --exclude='.git' addon-rustdesk-server/ ha-rustdesk-server/
cd ha-rustdesk-server

# 4. Initialise a new git history and push to the new GitHub repo
git init -b main
git add .
git commit -m "Initial commit: Home Assistant add-on for RustDesk Server"
git remote add origin https://github.com/SankeerthBoddu/ha-rustdesk-server.git
git push -u origin main
```

### Option B - Preserve history

```bash
git clone https://github.com/SankeerthBoddu/addon-rustdesk-server.git
cd addon-rustdesk-server
git checkout main
git pull origin main

git remote set-url origin https://github.com/SankeerthBoddu/ha-rustdesk-server.git
git push -u origin main
```

---

## Step 2 - Let GitHub Actions publish to GHCR

On `https://github.com/SankeerthBoddu/ha-rustdesk-server`:

1. Go to **Settings** -> **Actions** -> **General**.
2. Scroll to **Workflow permissions**.
3. Select **Read and write permissions**.
4. Tick **Allow GitHub Actions to create and approve pull requests**.
5. Click **Save**.

This lets the `deploy.yaml` workflow push container images to
`ghcr.io/sankeerthboddu/*-ha-rustdesk-server` using the built-in
`GITHUB_TOKEN` (no personal access token needed).

---

## Step 3 - Tag and publish the first release

On GitHub:

1. Go to **Releases** -> **Draft a new release**.
2. **Choose a tag**: type `v1.1.11-1` and click "Create new tag: v1.1.11-1 on
   publish".
3. **Target**: `main`.
4. **Release title**: `v1.1.11-1`.
5. **Description**: copy from `rustdesk-server/CHANGELOG.md` or leave blank
   (release-drafter will fill it on the next cycle).
6. Click **Publish release**.

Publishing the release triggers the **Deploy** workflow, which builds and
pushes three images:

- `ghcr.io/sankeerthboddu/amd64-ha-rustdesk-server:1.1.11-1`
- `ghcr.io/sankeerthboddu/aarch64-ha-rustdesk-server:1.1.11-1`
- `ghcr.io/sankeerthboddu/armv7-ha-rustdesk-server:1.1.11-1`

You can watch it on the **Actions** tab. When done, the three packages will
appear under **Packages** on your profile.

(Optional) Make the packages **public** so Supervisor does not need
authentication:

1. Your profile -> **Packages** -> click each `*-ha-rustdesk-server` package.
2. **Package settings** -> **Change visibility** -> **Public**.

---

## Step 4 - Add the repo to Home Assistant

1. Home Assistant UI -> **Settings** -> **Add-ons** -> **Add-on Store**.
2. Top-right menu (three dots) -> **Repositories**.
3. Paste `https://github.com/SankeerthBoddu/ha-rustdesk-server` and click
   **Add**, then **Close**.
4. Refresh the store - **RustDesk Server** now appears under your repository.
5. Click **Install**.

---

## Step 5 - Configure and start the add-on

1. Open the **RustDesk Server** add-on page.
2. **Configuration** tab - minimum recommended settings:

   ```yaml
   encrypted_only: true
   relay: rustdesk.your-domain.tld
   private_key: ""
   public_key: ""
   ```

   Leave `private_key` / `public_key` empty to let `hbbs` generate a fresh
   keypair on first boot.

3. **Info** tab -> enable **Watchdog** and **Start on boot**.
4. Click **Start**.
5. **Log** tab - you should see `hbbs` and `hbbr` start, and the generated
   public key be reported.
6. Copy the contents of `/addon_configs/<slug>/id_ed25519.pub` from your HA
   host (via File Editor or SSH add-on) - paste it into every RustDesk
   client as the **Key**, set **ID server** to your host, set **Relay
   server** to the value you used for `relay` above.

---

## Step 6 - Open router ports (if you want external access)

Forward these to the Home Assistant host:

| Port        | Required? | Purpose                                       |
| ----------- | --------- | --------------------------------------------- |
| 21115/tcp   | yes       | NAT type test, online status                  |
| 21116/tcp   | yes       | TCP hole punching and connection service      |
| 21116/udp   | yes       | ID registration and heartbeat                 |
| 21117/tcp   | yes       | Relay service                                 |
| 21118/tcp   | optional  | RustDesk Web Client (hbbs)                    |
| 21119/tcp   | optional  | RustDesk Web Client (hbbr)                    |
| 21114/tcp   | optional  | hbbs web console / API (Pro)                  |

---

## Step 7 (optional) - Archive or delete `addon-rustdesk-server`

Once `ha-rustdesk-server` is live and working:

1. Go to `https://github.com/SankeerthBoddu/addon-rustdesk-server/settings`.
2. Scroll to **Danger Zone**.
3. Either **Archive this repository** (recommended - keeps history,
   read-only) or **Delete this repository**.

---

## Troubleshooting

- **Deploy workflow fails with `permission_denied` on push to GHCR**
  Step 2 was skipped or incomplete. Re-check **Settings -> Actions ->
  General -> Workflow permissions**.
- **Home Assistant cannot pull the image**
  Either publish the release (Step 3) or set the GHCR packages to public
  (Step 3 optional bullet). If the image is still private, Supervisor will
  fall back to a local build - that also works, just slower.
- **CI fails on the first push**
  That is expected if you have not yet enabled workflow write permissions.
  Re-run the failed workflow after Step 2.

---

## What changed in this refactor (for your reference)

- Dropped deprecated `i386` architecture; added `amd64`, `aarch64`, `armv7`.
- Modern `config.yaml`: `startup`, `boot`, `host_network`, `options`,
  `schema`, `ports_description`, pinned version, pre-built `image`.
- Multi-architecture `Dockerfile` with preserved executable bits.
- `s6` services now `exec` their binaries and use modern `bashio` APIs.
- All French text translated to English; `fr.yaml` removed.
- Self-contained GitHub workflows (no external fork dependency).
- Added `CHANGELOG.md`, `repository.yaml`, release-drafter config.
- All URLs/labels/image paths point at `ha-rustdesk-server`.
