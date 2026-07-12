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

## [0.4.1] — 2026-07-13

One decision (#27) and one recorded fact, no rule changes — a **patch** by
CONTRIBUTING §4's discipline: nothing a conforming tool must do changes, and
the stability key stays `0.4`, which epubveri (v0.5.x) and epubsana (v0.2.0)
ship claiming today.

### Changed — FORMATS.md is no longer provisional

- **epubveri v0.5.0 (2026-07-11) shipped the first `--format json`; epubsana
  v0.2.0 followed.** By the provisional notice's own terms the stability
  guarantee began at that first implementation — the release records the fact
  (banner replaced with the implemented-as-of record; provisional references
  in CLI.md §3.6 and README updated). The guarantee was activated by the
  implementation, not by this release.

### Added — reference-implementation pointer (#27)

- FORMATS.md §2 points, **non-normatively**, at the envelope's reference Rust
  types: `epubveri::envelope`, generic over the two tool-owned slots, used by
  epubveri and epubsana — with the promotion trigger on the record:
  extraction to a `veripublica-envelope` crate the day a veripublica tool
  that does **not** depend on epubveri needs the envelope. The decision is
  CONTRIBUTING §2's promotion rule applied to implementation shapes; no new
  rule was needed, and none was added. (The type-shape review conventions was
  asked for lives on
  [epubveri#14](https://github.com/veripublica/epubveri/issues/14).)

## [0.4.0] — 2026-07-11

Three decisions (#23, #24, #25), one batch — the first shaped by adoption:
epubveri hit two spec wrinkles implementing v0.3's envelope, and epubsana
answered the discussion invite with the third. FORMATS.md **remains
provisional**: the stability guarantee still begins with the first
implementation.

### Changed — the severity vocabulary (#23)

- **`severity` becomes the five-value reserved set
  `fatal | error | warning | info | usage`** — epubcheck's vocabulary,
  completing the epubcheck-compatible `code` contract. A reserved value *set*:
  a tool emits only the values it has a concept of. The set is **closed** — a
  new value enters by issue + release, never through `data` (the corollary now
  recorded in CONTRIBUTING §2: *no value is invented for an unnamed need*).
- **On a `fix`/`operation` item, `severity` is inherited** — the severity of
  the finding the item addresses, verbatim from the detector, never how
  noteworthy the report line is. The transformer example's `RSC-016` fix now
  reads `"error"`.
- **`usage` items are always present in `json`**; suppression-by-default is a
  `human`-surface concern, and a display opt-in stays tool-owned.
- **A coded `fatal` finding makes its input `status: "problems"`, never
  `"error"`** — `"error"` keeps meaning *no report was possible*, never *the
  verdict was very bad*. CLI.md §6's exit-`2` wording follows: "unreadable",
  no longer "corrupt" — a verdict is not a failure.

### Changed — the exit-code / `status` threshold (#24)

- **The verifier's `0`/`1` line sits at error severity and above**: warning,
  info, and usage findings are reported, never flagged. `0` = every input
  processed, no error/fatal-severity findings; `1` = at least one remains.
- **The transformer's threshold defines its default goal** — no error- or
  fatal-severity findings remain — so a transformer's `0` means what a
  verifier's `0` means, by construction. An explicitly-requested lesser goal
  (epubsana's `--goal openable`) defines its own success condition in the
  tool's `--help`; under such a goal, exit `0` can coexist with error-severity
  findings in the report — the exit code answers the question the invocation
  asked. `--goal` itself stays tool-owned.
- **Deferred**: a shared `goal` field. One tool has the concept; promotion
  waits for the second, by issue.

### Added — item `outcome` (#25)

- **`outcome` is a shared item field**: `"applied" | "skipped" | "proposed"`.
  Required on `fix`/`operation` items, never present on `finding` items; under
  `dry_run: true` every item is `"proposed"`, demoting the top-level flag to a
  summary of the items — the two can never disagree. Declined on the record: a
  `"failed"` value (no tool has named the need); tool-owned `data.outcome`
  (a fact a consumer needs to read the report correctly cannot live in the
  half consumers cannot rely on); a normative line without a field (drift by
  another name).

### Non-normative, shipped on `main` since 0.3.0

- CONTRIBUTING.md §2 records the first principle's corollary for reserved
  value sets: **no value is invented for an unnamed need** — the protection
  against tomorrow's need is the extension path, not a blank value today.

## [0.3.0] — 2026-07-11

Two decisions (#20, #21), one batch. FORMATS.md's envelope is redesigned while
it is still provisional and free to change — which is the entire point of the
provisional notice.

### Changed — FORMATS.md, the envelope (#21)

- **`inputs[]` array replaces the single `input` string** — uniformly, an array
  even for one input, so every consumer writes one loop. Each element is a
  self-contained input object: `path`, `status`, `error` (why this input could
  not be processed), `output` (transformers: the path written — the fact the
  human output already prints), per-input `summary` and `items`.
- **Top-level `status` mirrors the exit code** by #16's aggregation; **`dry_run`**
  marks a `--dry-run` report of identical shape (#9's deferred half, resolved).
- **A usage error produces no envelope**: stderr text, empty stdout (#6). The
  envelope describes runs that happened.
- **`inputs[]` preserves command-line order.** Item fields are unchanged.
- **The standard/tool-owned boundary is written down**: the skeleton (envelope,
  input object, shared item fields) is standard; `summary` keys and
  `items[].data` are each tool's own, promoted to shared fields only when a
  second tool needs them — by an issue, not by drift.
- NDJSON was rejected on the record; pipeline-orchestration rationale for
  `output` was rejected as out of scope. FORMATS.md **remains provisional**:
  stability begins with the first implementation.

### Added — CLI.md §3.1 (#20)

- The `<semver>` printed by `-V` **MAY carry SemVer build metadata**
  (`+<short-hash>`, `.dirty`) pinning the exact source; unavailable (e.g. a
  crates.io build) → silent fallback to the plain version, MUST. One string
  everywhere: `-V`, a wasm `version()`, a demo footer.

### Non-normative, shipped on `main` since 0.2.0

- The site wears the family look: `assets/tokens.css` is a verbatim copy from
  [family-web](https://github.com/veripublica/family-web) (template v1),
  replacing the Cayman theme.
- FORMATS.md's sections are numbered, matching CLI.md and CONTRIBUTING.md.
- CONTRIBUTING.md carries three lines of front matter — the
  jekyll-optional-front-matter plugin blacklists it by name and would otherwise
  serve it as raw markdown.

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

[Unreleased]: https://github.com/veripublica/conventions/compare/v0.4.1...HEAD
[0.4.1]: https://github.com/veripublica/conventions/compare/v0.4.0...v0.4.1
[0.4.0]: https://github.com/veripublica/conventions/compare/v0.3.0...v0.4.0
[0.3.0]: https://github.com/veripublica/conventions/compare/v0.2.0...v0.3.0
[0.2.0]: https://github.com/veripublica/conventions/compare/v0.1.0...v0.2.0
[0.1.0]: https://github.com/veripublica/conventions/releases/tag/v0.1.0
