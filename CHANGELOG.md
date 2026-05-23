# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html). Because the deliverable is a single workflow JSON rather than a released library, "version" here corresponds to the `versionId` field inside the n8n workflow export.

## [Unreleased]

### Added

- Repository scaffolding: `README.md`, `LICENSE` (MIT), `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `SECURITY.md`, `CHANGELOG.md`.
- `.gitignore` and `.editorconfig` aligned with the n8n self-hosted layout.
- `.env.example` documenting the runtime variables for self-hosted n8n.
- `docs/architecture.md` with a mermaid flowchart matching the n8n graph, a sequence diagram of one lead-to-email round trip, and a "what lives where" table.
- `docs/getting-started.md` covering import into n8n Cloud and self-hosted setups, credential configuration, and a first dry run.
- `.github/` issue and pull-request templates.
- `.github/workflows/ci.yml` that validates the workflow JSON parses and that no secret patterns are committed.

### Changed

- Nothing. The workflow JSON has not been modified.

### Notes

- The current workflow corresponds to the export with `versionId` `1f1b2cbd-61d0-4df2-9128-3da22eb70dec`.

## How to update this file

When you change the workflow JSON or any documentation, add an entry under `[Unreleased]` describing what changed. When you cut a release, move the entries under a new heading like `## [1.0.0] - 2026-05-23` and start a fresh `[Unreleased]` section above it.
