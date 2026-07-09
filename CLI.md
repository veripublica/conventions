# veripublica CLI convention

**Version 0.1.0.** The command-line contract every veripublica tool follows. Key
words **MUST**, **SHOULD**, **MAY** are used in the
[RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) sense.

The goal: a user or a script that knows one veripublica tool already knows how
to drive the next — same flags, same output naming, same exit codes — and one
tool can consume another's output without special-casing.

---

## 1. Invocation

```
<tool> [OPTIONS] [<input>]
<tool> <subcommand> [OPTIONS] [<input>]
```

- A **single-purpose** tool (e.g. epubveri, epubsana) MAY have no subcommand: it
  performs its one action by default.
- A **multi-purpose** tool (e.g. epublift) SHOULD group actions under
  subcommands, and MAY still have a default action when none is given.

## 2. Input

- Every tool **MUST** accept its input via `-i, --input <PATH>`.
- A tool **SHOULD** also accept a single **positional** `<PATH>` as sugar, so
  `tool book.epub` works. `-i` wins if both are given.
- `-` **MAY** mean stdin where a tool supports streaming.

## 3. Reserved options

These names have the **same meaning in every tool**. A tool implements the ones
that apply to it and **MUST NOT** repurpose them for anything else.

| Option | Meaning |
| --- | --- |
| `-i, --input <PATH>` | Input file. Positional input SHOULD also be accepted. |
| `-o, --output <PATH>` | Output file (tools that produce one). See [§4](#4-output-and-file-safety). |
| `--format <human\|json>` | Output/report format. `human` is the default. `json` MUST follow [FORMATS.md](./FORMATS.md). A tool MAY offer extra formats (e.g. `ids`), but `human` and `json` are reserved. |
| `--dry-run` | Compute and report what would happen; change nothing on disk. |
| `-y, --yes` | Assume "yes" for every prompt; run non-interactively. |
| `-q, --quiet` | Suppress non-essential output. Errors still go to stderr. |
| `-v, --verbose` | Emit more detail. |
| `-V, --version` | Print `<tool> <semver>` and exit `0`. |
| `-h, --help` | Print help and exit `0`. |
| `--` | End of options; everything after is positional. |

Tools **MAY** add their own options and subcommands freely; the rule is only
that the reserved names above keep their defined meaning.

## 4. Output and file safety

For any tool that writes a transformed file:

- The tool **MUST NOT** modify the input in place. The original is always left
  untouched; the user decides whether to delete it afterwards.
- If `-o` is not given, the default output path **MUST** be
  `<input-stem>_<verb>.epub`, in the input's directory, where `<verb>` names the
  operation and is fixed per tool/subcommand — e.g. `_fixed` (epubsana repair),
  `_repaired` / `_meta` / `_v3.3` (epublift). This makes outputs predictable and
  self-describing.
- The output path **MUST NOT** be equal to the input path. If `-o` resolves to
  the input, the tool **MUST** refuse with an error (exit `2`) rather than
  overwrite.
- A tool **MAY** work on a private temporary copy internally, but this is an
  implementation detail; the guarantees above are what matter.

## 5. Streams

- The **primary result** (the `human` report, or the `json` document) goes to
  **stdout**.
- **Diagnostics, progress, and interactive prompts** go to **stderr**.
- This lets `<tool> --format json input.epub > out.json` capture clean,
  parseable output while messages still reach the user.

## 6. Exit codes

| Code | Meaning |
| --- | --- |
| `0` | Success. For a **verifier**: the input is valid/clean. For a **transformer**: the operation completed and the result meets its goal (e.g. the book is fully valid after repair, or was already). |
| `1` | Completed, but problems remain. For a **verifier**: the input is invalid (errors found). For a **transformer**: unresolved problems remain (e.g. the repairer could not clear every error). |
| `2` | The tool could not run: bad arguments, unreadable/corrupt input, an output-equals-input refusal, or an I/O failure. Nothing meaningful was produced. |

Rationale: `1` vs `0` lets a script branch on "is it actually good?"
(`tool book.epub && echo ok`), while `2` is reserved for "the tool itself
failed," so the two are never confused.

## 7. Versioning & conformance

- Tools use [SemVer](https://semver.org/). Before a tool's own `v1.0.0`, its CLI
  MAY still change.
- This convention is itself `0.x`: while below `1.0.0`, any rule here MAY change.
  See [CONTRIBUTING.md](./CONTRIBUTING.md#4-versioning).
- A tool SHOULD state the convention version it targets (in `--help`, README, or
  docs), e.g. *"conforms to veripublica conventions v0.1."* The version named
  MUST be one that has been tagged; a claim against `main` points at a moving
  document.
