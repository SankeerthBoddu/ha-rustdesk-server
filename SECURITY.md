# Security Policy

## Supported versions

Security fixes are made against the latest published Home Assistant App
release. Users should update to the latest release before reporting an issue
that may already have been fixed upstream or in the App packaging.

## Reporting a vulnerability

Please do **not** open a public GitHub issue with exploit details, private keys,
Home Assistant tokens, credentials, or other sensitive information.

If GitHub's **Report a vulnerability** option is available in this repository's
Security tab, use that private reporting path. If private vulnerability
reporting is not available, contact the maintainer through the GitHub profile
before publishing sensitive technical details so a private channel can be
agreed on.

Include, when relevant:

- Home Assistant App version and architecture.
- Home Assistant OS / Supervisor versions.
- RustDesk client versions.
- Whether the issue is in the App packaging, its startup/configuration logic,
  its container image, or the upstream RustDesk Server binaries.
- Minimal reproduction information that does not expose real credentials or
  private infrastructure.

This community project does not promise a formal response SLA, but reports that
could expose users or their remote-access infrastructure are prioritized.

## Upstream RustDesk vulnerabilities

This repository packages binaries produced by the upstream
[RustDesk Server project][rustdesk-server]. A vulnerability in `hbbs`, `hbbr`,
`rustdesk-utils`, or the RustDesk wire protocol should also be reported through
the upstream RustDesk project's security process.

This repository is the right place to report security problems specific to the
Home Assistant integration, including:

- App configuration or startup behavior.
- Container packaging and base-image integration.
- Release/download integrity checks.
- Network defaults or documentation that could cause unsafe deployment.
- GitHub Actions and release-pipeline behavior for this repository.

## Supply-chain controls

The App pins the upstream RustDesk Server version and verifies downloaded
release archives with SHA-256 digests taken from the upstream GitHub release
asset metadata. CI also builds both supported architectures and scans the
resulting image for fixable high/critical OS-package vulnerabilities.

Inherited library findings from the Home Assistant base image are reported in
CI so they remain visible even when they cannot be fixed directly in this
repository.

[rustdesk-server]: https://github.com/rustdesk/rustdesk-server
