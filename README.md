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
| [FORMATS.md](./FORMATS.md) | The machine-readable output (`--format json`): a shared envelope so one tool can consume another's output without bespoke parsing. **Provisional** — no tool emits it yet; the name is reserved, the shape is a design proposal. |
| [CONTRIBUTING.md](./CONTRIBUTING.md) | What belongs in this repository, how a change is proposed and decided, versioning, and how the tools keep up. |

## How a tool conforms

Conformance is defined precisely in [CLI.md §8](./CLI.md#8-conformance). In
brief, a conforming tool:

1. Satisfies every rule of [CLI.md](./CLI.md) that **applies to it** — which
   rules apply is decided by the table in §8, not by judgement calls.
2. Gives any reserved name it implements the meaning defined there (it may add
   its own long-form options and subcommands on top).
3. States the convention version it targets — in its `--help`, its README, or
   its docs (e.g. *"conforms to veripublica conventions v0.2"*) — and that
   version is **tagged**.

`--format json` is **not** required for conformance while
[FORMATS.md](./FORMATS.md) is provisional; a tool that has not implemented it
rejects the value rather than approximating it.

The convention specifies **behaviour, not implementation.** A tool may parse its
arguments by hand or with a library (e.g. `clap`); it only has to expose the
defined interface. Likewise it may be pure-Rust or anything else.

## Versioning

This convention is versioned with [SemVer](https://semver.org/). Current version:
**0.2.0** — while below `1.0.0`, any rule MAY change, and the stability boundary
is the **minor** version. This is deliberate: the contract is being settled
before any tool reaches its own `v1.0.0`, which is the cheapest time to settle
it. See [CONTRIBUTING.md](./CONTRIBUTING.md) for how a change is proposed,
decided, and released.

## Status

The rule set is settled at `0.2.0` — eighteen proposals discussed and decided
(the issues record each decision and its rationale). [FORMATS.md](./FORMATS.md)
remains provisional. The tools adopt sequentially — epubveri, then epubsana,
then epublift — each tracked by an issue on its own repository; the spec leads,
the tools follow, release by release.

## License

The veripublica conventions are licensed **[CC-BY-4.0](./LICENSE)** — free to
adopt and adapt, including outside veripublica, with attribution. A permissive
license is deliberate: the point of a convention is for others to follow it.
