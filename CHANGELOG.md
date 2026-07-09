# Changelog

All notable changes to the veripublica conventions are recorded here. The format
follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and the
convention is versioned with [SemVer](https://semver.org/) — see
[CONTRIBUTING.md](./CONTRIBUTING.md#4-versioning) for what a major, minor, or
patch bump means for a specification.

While the convention is below `1.0.0`, **any rule may change**, and the stability
boundary is the **minor** version: `0.1` → `0.2` may break anything.

## [Unreleased]

Nothing yet. Accepted proposals are batched here before a release; see the
[open issues](https://github.com/veripublica/conventions/issues).

## [0.1.0] — 2026-07-10

The first tagged version. It records the contract as it already stood, so that a
tool saying *"conforms to veripublica conventions v0.1"* points at a fixed
document instead of a moving `main`.

### Added

- **CLI.md** — invocation shape, the reserved options (`-i/--input`,
  `-o/--output`, `--format`, `--dry-run`, `-y/--yes`, `-q/--quiet`,
  `-v/--verbose`, `-V/--version`, `-h/--help`, `--`), output-file naming and file
  safety, stream separation, and exit codes `0`/`1`/`2`.
- **FORMATS.md** — the `--format json` envelope. **Provisional:** no tool emits
  it yet. See [#15](https://github.com/veripublica/conventions/issues/15).
- **CONTRIBUTING.md** — what belongs here, how a change is proposed, decided, and
  released, and how the tools are told about it.
- **CHANGELOG.md** — this file.

### Changed

- The convention is now versioned `0.1.0` rather than described as *"1.0
  (draft)"*. `1.0` was never a valid SemVer string, and the tools' claim to
  *"conform to v1"* pointed at a version that had never been tagged.
- A tool declaring conformance MUST name a **tagged** version (CLI.md §7).
- The envelope's `convention` field example is now `"0.1"`, matching the
  stability boundary while below `1.0.0`. The field's *definition* still says
  "major version" and is under discussion in
  [#12](https://github.com/veripublica/conventions/issues/12).

[Unreleased]: https://github.com/veripublica/conventions/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/veripublica/conventions/releases/tag/v0.1.0
