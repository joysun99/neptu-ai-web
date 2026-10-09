# Repository Guidelines

## Scope and repository boundaries

This is the public Neptu installation distribution repository (`joysun99/neptu-ai-web`). It currently provides public installation documentation and the destination for versioned Global/Site GitHub Release assets. Installation scripts and version queries must be usable anonymously; private GHCR image authentication is a separate prerequisite.

- `../neptu-global`: Global implementation/contracts and authoritative `deploy/portable/global-install.sh`.
- `../neptu-ai-site`: Site implementation and authoritative `deploy/portable/site-install.sh`.
- `../neptu-ai-images`: image build/scan/push, release evidence, version catalogs, installer bundle generation and GitHub publishing. Use `installer-release.sh` there; the public repository is not a second build system.
- `../neptu-ai-demo`: synthetic device/browser and installation acceptance harnesses.
- `../neptu-ai` and `../neptu-ai-docker`: read-only legacy references. Never edit, commit or deploy from them as part of this work.

## Public content policy

Everything committed or uploaded here becomes public. Only publish reviewed installation scripts, secret-free example configuration/templates, public guides, immutable image digest/version manifests and checksum lists. Never publish private business source trees, credentials/tokens, private keys/certificates, production configurations, user/runtime data, full logs, image archives or private test evidence.

Global/Site remain the only installer source owners. Extract scripts/templates from the exact source commits corresponding to verified published image cohorts; do not hand-edit distributed copies. The shared read-only version reader belongs to `../neptu-ai-images/tools/installer-catalog/reader.mjs` and is embedded into each source installer before image build/publication.

## Release workflow

Prepare, verify and publish with the image repository's Bash entry points. Require complete same-source per-project cohorts, matching runtime/installer versions, publication pullback receipts and digest pins. Keep Global and Site versions separately recorded; a combined bundle does not prove cross-project compatibility.

Publish a non-overwriting draft, round-trip verify every uploaded asset, then publish as a prerelease with latest=false after explicit RELEASE confirmation. Existing tags/releases must not be overwritten. Use this repository's own default-branch commit for Release tags; private publishing/source commits are provenance only. Do not change stable/latest or production eligibility automatically.

Release assets: global-install.sh and/or site-install.sh, fixed parameter text files, release-manifest.json, SHA256SUMS, INSTALL.md and installers.tar.gz with reviewed templates. Preserve source provenance, checksums and public recovery/version documentation. GHCR cleanup can remove image versions independently; historical Release metadata does not guarantee current image availability, and install must stop on failed pulls.

## Installation and management principles

Deployment hosts require Bash and Docker Engine/CLI, registry login, domains/DNS and required network/storage access. Node and other implementation dependencies run inside fixed tool images. Do not require host Node, pnpm, Git or gh for normal install/manage. Downloads and checksum verification can use existing trusted tools or a browser.

Install/configure stages prepare stopped stacks. Runtime mode selection and startup remain explicit. Version lookup must fail on missing/incompatible cohorts; do not silently mix independently latest images, bypass upgrade hashes/recovery gates or restart services. Never execute downloaded scripts through curl | bash.

## Local layout, checks and changes

README.md links public documentation under docs/. No application build/dependency manifest is currently defined. Do not add a framework or runtime dependencies just for distributing files.

For documentation changes, check links, examples, placeholder domains and git diff --check. Installer changes belong in source repositories and require affected Bash/Node tests plus image-repository bundle/query tests. Verify bash -n for scripts and preserve file checksums before publication. Use synthetic data for tests; do not use production secrets.

Review diffs and remove sensitive data before committing. Use scoped Conventional Commit subjects. Do not push or publish automatically; follow the user's current task authorization. Do not casually delete archives, reset Git history or force-push.
