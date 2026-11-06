# Changelog

All notable changes to this Home Assistant App are documented here.

## 1.1.16-4

### 1.1.16-4 Bug Fixes

- Fixed the App failing to start on 1.1.16-3: the s6 service scripts
  (`hbbs/run`, `hbbr/run`, `key-secret/up.real`) lost their executable
  bit when they were updated through the GitHub API, so s6 could not
  execute `key-secret/up.real` (`Permission denied`, exit 126) and the
  container stopped. The scripts are now made executable at build time
  in the Dockerfile, so a lost executable bit in git can no longer break
  a release.

### 1.1.16-4 Notes

- No RustDesk Server change — still bundles RustDesk Server `1.1.16`.

## 1.1.16-3

### 1.1.16-3 Build and Security

- Updated the Home Assistant base image from `21.0.1` to `21.0.10`,
  fixing the fixable high-severity OS-package findings (brotli, curl,
  jq, openssl) that Trivy reported on the previous base image.
- Updated the home-assistant/builder actions from `2026.06.0` to
  `2026.09.0`.

### 1.1.16-3 Maintenance Updates

- Updated `hadolint-action` to v3.5.0.
- Updated `markdownlint-cli2-action` to v25.
- Renamed the App configuration mount from `addon_config` to
  `app_config` and updated the remaining "Add-on" comments in the
  service scripts to "App".

### 1.1.16-3 Notes

- No RustDesk Server change — still bundles RustDesk Server `1.1.16`.

## 1.1.16-2

### 1.1.16-2 Build and Security

- Migrated from the retired legacy Home Assistant builder to the current
  BuildKit-based App build actions.
- Updated the Home Assistant base image to `21.0.1`.
- Added native `amd64` and `aarch64` CI builds and a signed multi-architecture
  release image.
- Added SHA-256 verification for downloaded RustDesk Server release archives.
- Added CI vulnerability scanning that blocks fixable high/critical OS-package
  findings and reports inherited high/critical library findings.
- Switched the App image to the generic multi-architecture
  `ghcr.io/sankeerthboddu/ha-rustdesk-server` package.

### 1.1.16-2 Release and Maintenance

- Made `config.yaml` the authoritative Home Assistant App release version so
  release tags match packaging revisions such as `1.1.16-2` exactly.
- Updated Renovate for the BuildKit/base-image layout and added a reminder to
  review upstream RustDesk asset digests with RustDesk version bumps.
- Relaxed stale issue handling and removed automatic locking of closed support
  threads.
- Added a security reporting policy and more actionable bug-report diagnostics.

### 1.1.16-2 Documentation

- Updated project terminology to Home Assistant App (formerly add-on).
- Added direct support links to the Home Assistant RustDesk community thread.
- Expanded Tailscale guidance, including LAN subnet routing versus direct
  `100.x` addressing and `Ready` status versus a complete session path.
- Clarified relay configuration, required ports, NGINX Streams, Cloudflare,
  persistence, backups, and common connection failures.
- Added a visible maintenance statement explaining how upstream updates are
  monitored and tested.

## 1.1.16-1

### 1.1.16-1 Runtime Changes

- Updated bundled RustDesk Server from `1.1.15` to `1.1.16`.
- Fixed the upstream long-offline peer status overflow issue.
- Included upstream dependency and UDP punch-hole security fixes.

### 1.1.16-1 Maintenance Updates

- Updated `markdownlint-cli2-action` to v24.
- Updated `actions/stale` to v11.
- Updated `hadolint-action` to v3.4.0.

## 1.1.15-1

### Runtime Changes

- Updated bundled RustDesk Server from `1.1.11-1` to `1.1.15`.
- Updated Home Assistant App base images to `v20.1.1` for `amd64` and
  `aarch64`.
- Reduced scheduled automation noise by switching maintenance workflows to
  weekly runs.

### Workflow Fixes

- Updated lock maintenance workflow to `dessant/lock-threads@v6`, resolving
  recent `Lock` workflow failures and Node.js runtime deprecation warnings.

### Maintenance Updates

- Updated CI and release workflow actions to current major versions for
  improved GitHub Actions compatibility.

## 1.1.11-1

### Breaking

- Replaced the deprecated `i386` architecture with modern ones:
  `amd64`, `aarch64`, and `armv7`. This resolves the Home Assistant warning
  *"only supports architectures and/or machines which are no longer supported
  by Home Assistant"*.
- Configuration schema reworked:
  - `ENCRYPTED_ONLY` environment variable replaced with an `encrypted_only`
    boolean option.
  - `relay` default is now empty; set it explicitly if needed.

### Added

- Multi-architecture support: `amd64`, `aarch64`, `armv7`.
- `host_network: true` for correct NAT-type detection and UDP hole punching.
- `ports_description` for clearer port explanations in the UI.
- `CHANGELOG.md`, `repository.yaml`, English-only translations.
- Self-contained GitHub Actions workflows (CI, deploy, stale, lock,
  release drafter) - no longer depend on a private workflows fork.

### Changed

- All documentation and in-app text translated to English.
- `hbbs` and `hbbr` now `exec` their binaries (cleaner signal handling under
  s6-overlay).
- Upgraded bashio usage: `bashio::config.true`, `bashio::config.has_value`,
  `bashio::exit.nok`.

### Fixed

- Binary download now works for all supported architectures.
- Key validation errors abort the service via `bashio::exit.nok` instead of
  halting the container abruptly.
