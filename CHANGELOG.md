# Changelog

All notable changes to the veripublica conventions are recorded here. The format
follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and the
convention is versioned with [SemVer](https://semver.org/) — see
[CONTRIBUTING.md](./CONTRIBUTING.md#4-versioning) for what a major, minor, or
patch bump means for a specification.

While the convention is below `1.0.0`, **any rule may change**, and the stability
boundary is the **minor** version: `0.1` → `0.2` may break anything.

## [Unreleased]

Decisions that have been taken but not yet written into the documents are the
**open issues labelled [`accepted`](https://github.com/veripublica/conventions/issues?q=is%3Aopen+label%3Aaccepted)**.
That label is the queue; this section is filled in by the pull request that ships
them. Listing them here as well would be a second copy to keep in step, and it
would drift.

## [0.2.0] — 2026-07-10

Eighteen proposals (#1–#18), discussed and decided one by one, written into the
documents in a single batch. Each issue's closing **Decision** comment records
what was decided, why, and what was rejected. Breaking changes are listed as
such; under `0.x` the stability boundary is the minor version, so `0.1` → `0.2`
may break anything — and does.

### Breaking

- **Input is `-i, --input` only; positional paths are forbidden** (#16). A
  transformer takes exactly one input; a verifier may repeat `-i`. `-` (stdin)
  is removed; `--` is accepted-and-ignored, nothing more (#3).
- **An unrecognized token is a usage error** — option-like or bare (#3). The
  only bare token a tool accepts is a subcommand name from its fixed vocabulary.
- **An option-argument outside the option's defined set is a usage error**,
  never a silent fallback — reserved-but-unimplemented `--format json` included
  (#7).
- **A single-valued option given twice is a usage error**, never last-wins
  (#18).
- **`--format` is mandatory** in every tool and every subcommand, supporting at
  least `human` (#8).
- **An existing output file is never silently overwritten**: refuse with exit
  `2`; `-f, --force` (new reserved flag) permits replacement and never lifts the
  output-equals-input refusal; overwrite permission is never obtained by prompt
  (#4).
- **A tool never prompts when stdin is not a TTY**; when it cannot obtain a
  decision it stops with exit `2`, naming the unblocking option (#5).
- **Short flags come only from the closed reserved set**
  (`-i -o -f -q -v -y -h -V`); every other option is long-form only (#17).
- **Required syntaxes**: `--name=VALUE`, attached `-nVALUE`, and bundling of
  value-less short flags, with POSIX semantics for value-taking flags in a
  bundle (#11).
- **Exit `2` redefined for multi-input runs**: at least one input could not be
  processed; every input is still processed and reported (#16).

### Added

- **Conformance defined** (CLI.md §8): applicability table, no levels, claims
  name a tagged stability key (#8).
- **Help baseline** (CLI.md §7): synopsis + complete options list MUST;
  examples, exit codes, conformance line SHOULD; `-h` ≡ `--help`; canonical
  reserved-option descriptions; all CLI text in English — localization belongs
  to GUI layers (#2).
- **Color rules** (CLI.md §5): per-stream TTY gating, `NO_COLOR`, `--color`
  precedence; `json` never colorized (#1).
- **`--dry-run` semantics** (CLI.md §3.7): validates everything, fails
  identically, writes nothing; its `json` shape is deferred to the envelope
  redesign (#9).
- **`-v` is a boolean**: no levels, repetition idempotent, `-q`/`-v` last-wins
  (#10).
- **Prior art section** (CLI.md §10): POSIX, GNU, no-color.org — correctly
  labelled, with the deliberate departures listed and linked (#14).
- **Repetition table** (CLI.md §3.4) unifying #10, #16, #18.

### Changed

- **FORMATS.md is provisional** (#15): no tool emits the envelope; the stability
  guarantee begins with the first implementation. Known-open: multi-input
  `input`, dry-run shape.
- **`convention` field carries the stability key** — `"0.1"` while `0.x`, `"1"`
  from `1.0.0`; definition and example now agree (#12).
- **Default output naming generalised** to `<input-stem>_<verb><ext>`; the
  extension-changing transformer class defined (#13).
- **RFC 8174 cited alongside RFC 2119** (BCP 14 formula) (#14).

### Fixed

- **CONTRIBUTING.md** contradicted itself: §3 closed an issue on decision, while
  §4 had the release pull request close it. Resolved in favour of §4 — `accepted`
  issues stay open until a release writes them down, so the open `accepted` label
  is the queue that batching depends on. `declined` issues close on decision.

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

[Unreleased]: https://github.com/veripublica/conventions/compare/v0.2.0...HEAD
[0.2.0]: https://github.com/veripublica/conventions/compare/v0.1.0...v0.2.0
[0.1.0]: https://github.com/veripublica/conventions/releases/tag/v0.1.0
