# veripublica conventions

**Shared conventions every [veripublica](https://github.com/veripublica) tool
follows** — so the tools compose cleanly, and users (and other programs) don't
have to re-learn each project's flags, exit codes, output names, or machine
format.

veripublica ships a pipeline of small, single-purpose EPUB tools:
**[epubveri](https://github.com/veripublica/epubveri)** (verify) →
**[epubsana](https://github.com/veripublica/epubsana)** (repair) →
**epublift** (modernize/optimize). Because they chain — one tool's output is the
next tool's input — a common contract is worth more than each tool's local
cleverness. This repository is that contract.

## Scope

These conventions govern the command-line **tools** — epubveri, epubsana,
epublift — that users invoke and that consume each other's files. The family
also includes shared pure-Rust **libraries** (styloria — CSS; schemora —
XPath/Schematron) that the tools build on. The CLI and output rules here don't
apply to libraries; libraries share only the house license and pure-Rust
conventions. Any library-level conventions (API shape, error types, position
reporting) would live in a separate document if the need arises.

## Documents

| Document | Covers |
| --- | --- |
| [CLI.md](./CLI.md) | Invocation shape, the reserved options every tool shares, output-file naming, and exit codes. |
| [FORMATS.md](./FORMATS.md) | The stable machine-readable output (`--format json`): a shared envelope so one tool can consume another's output without bespoke parsing. |

## How a tool conforms

A conforming tool:

1. Implements the reserved options from [CLI.md](./CLI.md) with their defined
   meanings (it may add its own options and subcommands on top).
2. Uses the shared exit-code and output-naming rules.
3. Emits the shared envelope for `--format json`.
4. States the convention version it targets — in its `--help`, its README, or
   its docs (e.g. *"conforms to veripublica conventions v1"*).

The convention specifies **behaviour, not implementation.** A tool may parse its
arguments by hand or with a library (e.g. `clap`); it only has to expose the
defined interface. Likewise it may be pure-Rust or anything else.

## Versioning

This convention is versioned with [SemVer](https://semver.org/). A **major**
bump is a breaking change to the contract; tools declare the major version they
target. Current version: **1.0 (draft)** — being adopted across the tools before
any of them reaches its own `v1.0.0`, which is the cheapest time to settle it.

## Status

Draft, actively adopted. epubsana is the reference implementation. epubveri and
epublift are aligned incrementally, backward-compatibly where they're already
released (add aliases, don't break existing invocations), tracked via GitHub
issues.

## License

The veripublica conventions are licensed **[CC-BY-4.0](./LICENSE)** — free to
adopt and adapt, including outside veripublica, with attribution. A permissive
license is deliberate: the point of a convention is for others to follow it.
