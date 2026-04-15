# Changelog

All notable changes to this add-on are documented here.

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

- All documentation and in-add-on text translated to English.
- `hbbs` and `hbbr` now `exec` their binaries (cleaner signal handling under
  s6-overlay).
- Upgraded bashio usage: `bashio::config.true`, `bashio::config.has_value`,
  `bashio::exit.nok`.

### Fixed

- Binary download now works for all supported architectures.
- Key validation errors abort the service via `bashio::exit.nok` instead of
  halting the container abruptly.
