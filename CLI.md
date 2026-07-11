# veripublica CLI convention

**Version 0.4.0.** The command-line contract every veripublica tool follows. The
key words **MUST**, **MUST NOT**, **SHOULD**, and **MAY** are to be interpreted as
described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) and
[RFC 8174](https://www.rfc-editor.org/rfc/rfc8174) when, and only when, they
appear in all capitals.

The goal: a user or a script that knows one veripublica tool already knows how to
drive the next — same flags, same output naming, same exit codes — and one tool
can consume another's output without special-casing.

Two principles run through every rule below:

- **Data never takes its meaning from its position.** Every path and value is
  named by the option that carries it. The only bare word a tool understands is a
  subcommand — a verb from a closed set the parser can validate.
- **When intent is ambiguous, stop loudly.** A tool never guesses, never silently
  drops, corrects, or picks a survivor among conflicting instructions.

---

## 1. Invocation

```
<tool> [OPTIONS]
<tool> <subcommand> [OPTIONS]
```

- A **single-purpose** tool (e.g. epubveri, epubsana) MAY have no subcommands: it
  performs its one action by default.
- A **multi-purpose** tool (e.g. epublift) SHOULD group actions under
  subcommands, and MAY still have a default action when none is given.
- The **only bare token** a tool accepts is a subcommand name from its own fixed
  vocabulary. Every other bare token is a usage error (exit `2`): a path or a
  value MUST NOT be accepted positionally.

## 2. Input

- A tool that takes a user-supplied input **MUST** accept it via
  `-i, --input <PATH>`, and **MUST NOT** accept a positional path.
- What the path may name — a file, a directory, an archive — is the tool's own
  business, documented in its help.
- A **transformer** (a tool that writes a transformed output file) takes
  **exactly one** input. A second `-i` is a usage error: exit `2`, saying how
  many inputs were expected and how many were given. The tool MUST NOT silently
  keep the last.
- A **verifier** (a tool that reads and reports, writing no output file) MAY
  accept more than one input, by repeating `-i`. See [§6](#6-exit-codes) for how
  the exit code aggregates.
- `-` has no special meaning; there is no stdin input. (An EPUB is a ZIP; reading
  a ZIP requires seeking, and stdin does not seek.)
- Batch work belongs to the shell, which already solves it:

  ```
  find . -name '*.epub' -print0 | xargs -0 -n1 epubsana -i
  ```

## 3. Options

### 3.1 Reserved options

These names have the **same meaning in every tool**. A tool implements the ones
that apply to it (see [§8](#8-conformance)) and **MUST NOT** repurpose them.
Reserved names bind **exact spellings**, not prefixes: `--image-format` does not
collide with `--format`. The one-line descriptions below are canonical — help
text SHOULD reproduce them verbatim (see [§7](#7-help)).

| Option | Meaning |
| --- | --- |
| `-i, --input <PATH>` | The input. The only input form; positional paths are not accepted. |
| `-o, --output <PATH>` | Where to write the output. See [§4](#4-output-and-file-safety) for the default and the safety rules. |
| `-f, --force` | Permit replacing existing output files. Never lifts the output-equals-input refusal. |
| `--format <FORMAT>` | Report format. `human` (the default) is always supported; `json` is reserved for [FORMATS.md](./FORMATS.md). |
| `--dry-run` | Report what would happen; change nothing on disk. |
| `-y, --yes` | Assume "yes" for every prompt; run non-interactively. Not permission to overwrite files — that is `-f`. |
| `-q, --quiet` | Suppress non-essential output. Errors still go to stderr. |
| `-v, --verbose` | Emit more detail. Boolean: there are no verbosity levels. |
| `-V, --version` | Print `<tool> <semver>` to stdout and exit `0`. |
| `-h, --help` | Print help to stdout and exit `0`. See [§7](#7-help). |
| `--` | Not used. A tool MAY accept and ignore it; it MUST NOT give it any other meaning. |

The `<semver>` printed by `-V` MAY carry **SemVer build metadata** pinning the
exact source: `+<short-hash>`, with `.dirty` appended when the working tree had
uncommitted changes at build time — `epubveri 0.4.4+ca06641`. Build metadata is
part of a valid SemVer string and is ignored for precedence, so the reserved
shape is unchanged for every consumer. When the hash is unavailable (notably a
crates.io tarball build, which has no `.git`) the tool **MUST** fall back
silently to the plain version: a published release is pinned by its tag; the
hash pins the builds *between* tags. One string, everywhere: the CLI's `-V`, a
wasm crate's `version()`, and a demo page's footer print the same value from
the same source.

### 3.2 The short-flag set is closed

Short flags are drawn **only** from the reserved set above. Every other option is
**long-form only** (`--quality`, `--profile`, `--goal`, …).

- Extending the set is a convention change — an issue on this repository — never
  a tool's own call. The set stays closed *because* nobody can open it locally.
- A tool's existing option name is changed only when it collides with the
  reserved set — never for taste. A long option that collides with nothing stays
  exactly as shipped.

### 3.3 Accepted syntaxes

Every tool, hand-rolled or library-parsed, accepts the same spellings:

- A long option with a value: both `--name VALUE` and `--name=VALUE` **MUST** be
  accepted.
- A short option with a value: both `-n VALUE` and the attached `-nVALUE`
  **MUST** be accepted.
- Short flags that take no value **MUST** be accepted bundled: `-qv` ≡ `-q -v`.
- A value-taking flag inside a bundle follows plain **POSIX semantics**: the
  first value-taking flag consumes the remainder of the token — or, if the
  remainder is empty, the next argument — as its value. `-iv` therefore means
  `-i v` in every tool. A tool MUST NOT give a bundle any other interpretation.
- Documentation and examples **MUST NOT** show a value-taking flag inside a
  bundle. The convention parses `tar -xzvf` style consistently; it does not
  teach it.
- The token following an option that takes a value is **always** that value, and
  is never re-parsed as an option: `-i -q.epub` names the file `-q.epub`.

### 3.4 Repetition

| Given more than once | Behaviour |
| --- | --- |
| A boolean flag (`-v -v`, `-qq`) | Same as once. MUST NOT be an error. |
| Opposing booleans (`-q … -v`) | The last one on the command line wins. |
| A single-valued option (`--format x --format y`) | Usage error, exit `2` — two answers to one question; the tool does not guess. |
| A multi-valued option (a verifier's `-i`) | Accumulates. |

> Implementation note: `clap`'s default boolean errors on repetition; declare
> repeatable booleans with `ArgAction::Count` and read them as `> 0`.

### 3.5 Unrecognized tokens and values

- A token that is not a recognized option, subcommand, or option-value is a
  **usage error**: exit `2`, a short message on stderr that points at `--help`
  (see [§5](#5-streams-prompts-and-color)). A tool MAY suggest a near match
  (*"did you mean `-V`?"*).
- An option-argument outside the option's defined set is a **usage error**: exit
  `2`, never a silent fallback or correction. The message **MUST** list the
  values the tool does support. Value names are lowercase and matched exactly.
- This applies to reserved-but-unimplemented values: a tool that has not
  implemented `json` **MUST** reject `--format json`. Reserving a name is not
  supporting it.

### 3.6 `--format`

- Every tool — and every subcommand — **MUST** accept `--format`, supporting at
  least `human`, which is the default.
- `human` output is for people and MAY change freely. `json` is reserved for the
  shared machine format ([FORMATS.md](./FORMATS.md), currently provisional).
  Tools MAY add their own formats (e.g. epubveri's `ids`).

### 3.7 `--dry-run`

- **MUST NOT** write, create, truncate, or delete any file.
- **MUST** perform every check the real run performs before doing work, and fail
  identically — the [§4](#4-output-and-file-safety) refusals included.
  `tool --dry-run … && tool …` must never surprise on the second half; a dry run
  that "succeeds" where the real run would refuse is a false plan.
- Exit code: `0` — the real run would have nothing to change; `1` — actions
  would be performed or problems would remain; `2` — the invocation could not
  run.

## 4. Output and file safety

For any tool that writes files:

- The tool **MUST NOT** modify the input in place. The original is always left
  untouched; deleting it afterwards is the user's decision, never the tool's.
  (A tool MAY work on a private temporary copy internally; that is an
  implementation detail.)
- If `-o` is not given, the default output path **MUST** be
  **`<input-stem>_<verb><ext>`**, in the input's directory, where:
  - `<verb>` names the operation and is fixed per tool/subcommand — `_fixed`
    (epubsana), `_repaired` / `_meta` / `_v3.3` (epublift);
  - `<ext>` is the natural extension of the **output** — usually the input's,
    but not necessarily; compound extensions (`_v3.3.kepub.epub`) are fine.
  - An **extension-changing transformer** — one whose operation *is* the format
    change (`import book.pdf` → `book.epub`) — MAY omit `<verb>`: the changed
    extension is already self-describing and cannot collide with the input.
  - The two properties this rule guarantees, for judging future cases:
    **predictability** (the user can name the output before running the tool)
    and **non-collision with the input**.
- The output path **MUST NOT** equal the input path. If `-o` resolves to the
  input, the tool **MUST** refuse: exit `2`. `-f` does **not** lift this.
- A tool **MUST NOT** silently overwrite **any** existing file it is about to
  write — the primary output or an auxiliary one (a report). It **MUST** refuse
  (exit `2`) with a message naming both the path and the way through:
  `error: 'book_fixed.epub' exists; use -f to replace it`.
- Overwrite permission is **never** obtained by prompt; only `-f` grants it.
  (A prompt would be answered by `-y` — and `-y` MUST NOT imply `-f`.)

## 5. Streams, prompts, and color

**Streams.**

- The **primary result** (the `human` report, or the `json` document) goes to
  **stdout**. Diagnostics, progress, warnings, and prompts go to **stderr**.
- Requested help (`-h`, `--help`) and version (`-V`, `--version`) are the
  primary result: **stdout**, exit `0`.
- A **usage error** goes to **stderr** and exits `2`: a short message naming the
  problem, plus a pointer to `--help` — never the full help text.
- A help request **short-circuits**: if `-h`/`--help` appears as an option token
  anywhere on the line, the tool prints help and exits `0`, even if the rest of
  the line is malformed. If both help and version are requested, help wins.
- A bare invocation is not a special case: missing required input is an ordinary
  usage error (`error: missing required -i; see --help`), not a help dump.
- Errors SHOULD begin `error: `; warnings SHOULD begin `warning: `.
- All user-facing text — help, errors, warnings, reports — is **English**.
  Localization belongs to GUI layers built on top of the CLI, not to the CLI.

**Prompts.**

- A tool **MUST NOT** prompt when **stdin is not a TTY**.
- When a tool needs a decision it cannot obtain, it **MUST** stop: exit `2`,
  with a message on stderr naming the option that would let it proceed
  (`--yes`, `--auto-safe`). It **MUST NOT** silently assume "no" and return an
  exit code that looks like an ordinary result.
- A tool **MUST NOT** prompt when `-y, --yes` is given; it proceeds as though
  the user accepted.
- A tool MAY have no prompts at all; these rules bind only tools that prompt.

**Color.**

- A tool MAY colorize `human` output. A tool that does:
  - MAY color a stream **only while that stream is a TTY** — checked per stream
    (stdout and stderr each on their own);
  - **MUST** disable color everywhere when the `NO_COLOR` environment variable
    is set to any value;
  - **MUST NOT** ever colorize `--format json` output.
- A tool MAY offer `--color <auto|always|never>`; `auto` is the default and
  means the rules above. Precedence: an explicit `--color always|never` on the
  command line > `NO_COLOR` > `auto`.
- Prompting keys on **stdin**'s TTY-ness; color keys on the TTY-ness of **the
  stream being written**. Different questions, different streams.

## 6. Exit codes

| Code | Meaning |
| --- | --- |
| `0` | Success. For a **verifier**: every input processed; no error- or fatal-severity findings. For a **transformer**: the operation completed and its goal was met. |
| `1` | Completed, but problems remain. For a **verifier**: every input processed; at least one error- or fatal-severity finding remains. For a **transformer**: the goal was not met — declined fixes, or defects the tool cannot fix, remain. |
| `2` | The tool could not run, or could not process at least one input: usage errors, unreadable input, the [§4](#4-output-and-file-safety) refusals, an unanswerable prompt ([§5](#5-streams-prompts-and-color)), an I/O failure. |

- **The `0`/`1` line sits at error severity and above.** A finding below it —
  warning, info, usage ([FORMATS.md §1.3](./FORMATS.md#13-item-fields)) — is
  worth *reporting*, not worth *flagging*: it appears in the report and never
  moves the exit code. A script that wants to fail on warnings is asking a
  different question, and that is a tool-owned option's job, never the
  default's.
- **A verdict is not a failure.** For a verifier, corruption it can still
  produce a report on is a *verdict* — exit `0` or `1` by the threshold above,
  the findings in the report (a `fatal`-severity finding included). Exit `2`'s
  "could not process" means **no report was possible** for that input: a
  missing file, an unreadable one, an I/O failure.
- **A transformer's default goal is the verifier's threshold** — no error- or
  fatal-severity findings remain — so the two tools' `0` agrees by
  construction. A tool offering an explicitly-requested **lesser** goal
  (epubsana's `--goal openable`) defines that goal's success condition in its
  own `--help`. Under such a goal, exit `0` can coexist with error-severity
  findings in the report: the exit code answers the question the invocation
  asked. The goal knob itself stays the tool's own option, never the
  convention's.
- A multi-input verifier **MUST** process every input it was given and report on
  each, even when an earlier one could not be processed. Stopping at the first
  failure throws away work already done and makes a CI job discover its broken
  inputs one run at a time. Exit `2` for a multi-input run therefore means *"at
  least one input could not be processed"* — the reports for the others still
  appear. (`grep` behaves this way.)
- Rationale: `1` vs `0` lets a script branch on "is it actually good?", while
  `2` means "the tool itself could not do its job" — and a tool that silently
  answered its own prompt, or silently skipped an input, would be reporting the
  first while meaning the second. The two are never conflated.

## 7. Help

- `-h` and `--help` print the **same** text.
- Help **MUST** contain:
  - a **usage synopsis** — the invocation form(s);
  - the **complete options list** — every option the tool accepts, one line
    each; value-taking options show their metavar (`--goal <openable|valid>`);
    defaults are stated; `-v` and `-V` are thereby visible side by side;
  - for a subcommand tool: the **subcommand list**, with per-subcommand help via
    `tool <sub> --help`, under these same rules.
- Help **SHOULD** contain:
  - an **EXAMPLES** section — at least one common invocation, with a comment;
  - an **EXIT CODES** summary, in the tool's own terms (*"0 — the book is valid
    after repair (or was already)"*);
  - a **conformance line** naming a tagged version: *"Conforms to veripublica
    conventions v0.4."*
- Reserved options SHOULD be described with the canonical one-liners from
  [§3.1](#31-reserved-options), verbatim — read once, recognized in every tool.
- **Help is the reference; the error message is the front line.** Nobody reads
  manuals, everybody reads errors: every usage error names the problem, names
  the way through (`--help`, `-f`, `--yes`, the supported values), and lands the
  user here.

## 8. Conformance

A tool **conforms to veripublica conventions vX** when both hold:

1. it satisfies every rule above that **applies to it**; and
2. any reserved name it implements carries the **meaning defined here**.

There are no conformance levels. "Applies to it" is decided by this table:

| Rules | Required when |
| --- | --- |
| `-h`, `-V`, `--format` (§3.6), accepted syntaxes (§3.3), repetition (§3.4), unrecognized-token handling (§3.5), streams (§5), exit codes (§6), help (§7) | **Always** |
| `-i` and the no-positional rule (§2) | The tool takes a user-supplied input |
| `-o`, output naming and file safety (§4), `-f` | The tool writes files |
| `-y` and the prompt rules (§5) | The tool prompts |
| `--format json` ([FORMATS.md](./FORMATS.md)), `--dry-run`, `-q`, `-v`, `--color` | **Never** — but when implemented, with the defined meaning |

A verifier needs no `-o` and no `--dry-run`; a tool with no prompts needs no
`-y`. That is why there are no levels: "full conformance" would name a target no
tool should even want to reach.

The claim — *"conforms to veripublica conventions v0.4"* — names the convention's
**stability key** (see [§9](#9-versioning)), and a tag with that prefix (e.g.
`v0.4.0`) MUST exist: a claim against `main`, or against an untagged version,
points at a moving document and asserts nothing.

## 9. Versioning

- Tools use [SemVer](https://semver.org/). Before a tool's own `v1.0.0`, its CLI
  MAY still change.
- This convention is versioned with SemVer and is itself `0.x`: while below
  `1.0.0`, any rule MAY change, and the **stability boundary is the minor
  version** (`0.1` → `0.2` may break anything). From `1.0.0` on, the boundary is
  the major version. The version prefix at that boundary — `0.4` today, `1`
  after `1.0.0` — is the convention's **stability key**: the string tools claim
  ([§8](#8-conformance)) and machine output carries
  ([FORMATS.md](./FORMATS.md)).
- How changes are proposed, decided, batched, and released — and what a major,
  minor, or patch bump means for a specification — is
  [CONTRIBUTING.md](./CONTRIBUTING.md)'s subject.

## 10. Prior art — and departures

**No RFC governs command-line syntax.** RFC 2119 and RFC 8174, cited in the
preamble, define the requirement keywords and nothing else in this document. The
rules above descend from two non-RFC sources, and from one de facto convention:

- **POSIX** (IEEE Std 1003.1, Utility Conventions) — short-option syntax and
  option-argument semantics, including the bundle rules §3.3 adopts;
- the **GNU Coding Standards** — long options, `--help`/`--version`, and the
  registry model behind §3.1: a table of names kept consistent across programs;
- **[no-color.org](https://no-color.org/)** — the `NO_COLOR` convention §5
  adopts; a de facto convention, not a standard.

Where this convention departs from those sources, it departs deliberately; each
departure was argued in the linked issue:

- **Input is named, never positional** (§2; [#16]) — against GNU's advice that
  ordinary arguments be the input files.
- **`--format` is required, not merely registered** (§3.6; [#8]) — GNU's option
  table never even fixes its meaning.
- **A repeated single-valued option is an error** (§3.4; [#18]) — against the
  getopt tradition of letting the last occurrence win.
- **No stdin `-`, and `--` is accepted-and-ignored** (§2, §3.1; [#16], [#3]) —
  POSIX's operand machinery removed along with the operands themselves.
- **Short flags come only from a closed set** (§3.2; [#17]) — stricter than
  either source imagines.

The recurring justification: POSIX and GNU serve hundreds of independent
programs, where uniformity can only ever be advisory. These are a handful of
tools in one pipeline with one owner, where explicitness and loud failure are
worth more than matching a reflex.

[#3]: https://github.com/veripublica/conventions/issues/3
[#8]: https://github.com/veripublica/conventions/issues/8
[#16]: https://github.com/veripublica/conventions/issues/16
[#17]: https://github.com/veripublica/conventions/issues/17
[#18]: https://github.com/veripublica/conventions/issues/18
