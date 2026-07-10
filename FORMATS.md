# veripublica machine-output format

**Version 0.2.0 — PROVISIONAL.**

> **No tool emits this envelope yet.** This document is a design proposal, not an
> observed contract: its shape MAY change without a major (or, while `0.x`,
> minor) version bump **until the first tool ships it** — the stability
> guarantee below begins at that first implementation, not before. Two design
> questions are known to be open and MUST be resolved before this notice comes
> off:
>
> 1. **Multiple inputs.** A verifier may take repeated `-i`
>    ([CLI.md §2](./CLI.md#2-input)); the envelope's `input` field is a single
>    string and cannot describe such a run.
> 2. **Dry runs.** A `--dry-run` invocation under `--format json` must be
>    distinguishable from a real one, and identical in shape
>    ([CLI.md §3.7](./CLI.md#37---dry-run)).
>
> The `json` format name is reserved by
> [CLI.md §3](./CLI.md#3-options) regardless: a tool that has not implemented it
> **rejects** `--format json` rather than falling back to `human`.

This document specifies the JSON a tool will emit under `--format json`, so one
veripublica tool (or any external program — Sigil, Calibre, a CI job) can consume
another's output without bespoke parsing.

`human` format is for people and MAY change freely. `json`, once implemented, is
a contract: its shape is stable within the convention's stability boundary
([CLI.md §9](./CLI.md#9-versioning)).

---

## The envelope

Every `--format json` invocation prints **one** JSON object to stdout:

```json
{
  "tool": "epubsana",
  "tool_version": "0.2.0",
  "convention": "0.1",
  "input": "book.epub",
  "status": "problems",
  "summary": { "errors_before": 150, "errors_after": 0, "applied": 2, "skipped": 0 },
  "items": [
    {
      "type": "fix",
      "code": "RSC-016",
      "rule": "htm.entity.undeclared",
      "severity": "info",
      "location": "OEBPS/ch1.xhtml",
      "message": "Mapped 774 undeclared HTML entities to characters",
      "data": { "occurrences": 774 }
    }
  ]
}
```

### Envelope fields

| Field | Type | Meaning |
| --- | --- | --- |
| `tool` | string | The tool's name (e.g. `"epubveri"`). |
| `tool_version` | string | The tool's SemVer. |
| `convention` | string | The convention's **stability key**: the version prefix at which stability is guaranteed — `"0.1"` while the convention is `0.x`, `"1"` from `1.0.0` on ([CLI.md §9](./CLI.md#9-versioning)). Compare with string equality; there is nothing finer to parse. |
| `input` | string | The input path as given. **Known-insufficient** for multi-input runs; see the provisional notice. |
| `status` | string | `"ok"` (nothing wrong / goal met), `"problems"` (findings remain), or `"error"` (the tool could not run — see [exit codes](./CLI.md#6-exit-codes)). Mirrors the exit code: `ok`→0, `problems`→1, `error`→2. |
| `summary` | object | Tool-specific counts (small, flat). Optional. |
| `items` | array | The findings / fixes / operations. May be empty. |

### Item fields

Each item is an object. These fields are **shared** — a consumer can rely on
them across tools — and a tool MAY add more under `data`:

| Field | Type | Meaning |
| --- | --- | --- |
| `type` | string | Item kind: `"finding"` (verifier), `"fix"` (repairer), `"operation"` (transformer). |
| `code` | string | The stable, tool-facing code — for EPUB tools, the epubcheck-compatible message ID (e.g. `"RSC-005"`). |
| `rule` | string | Optional finer sub-code (e.g. `"ncx.ids.invalid_ncname"`). |
| `severity` | string | `"error"`, `"warning"`, or `"info"`. |
| `location` | string | Container-relative path the item concerns, if any. |
| `position` | object | `{ "line": N, "column": N }` (1-indexed), if known. |
| `message` | string | Human-readable one-liner. |
| `data` | object | Tool-specific extra fields (counts, before/after values, …). |

Fields that don't apply MAY be omitted. Consumers MUST ignore unknown fields
(so tools can extend `data` without breaking anyone).

## Guarantees — from the first implementation on

- Exactly one JSON object on stdout; nothing else on stdout in `json` mode. The
  output is never colorized ([CLI.md §5](./CLI.md#5-streams-prompts-and-color)).
- Stable within the convention's stability boundary: fields are not removed or
  repurposed; new optional fields MAY be added.
- The envelope is shared; the meaning of `items[].data` is each tool's own and
  is documented in that tool's docs.

Until the first implementation ships, none of the above binds anyone — that is
what **provisional** means. It is written in contract language so that adopting
it, when the time comes, is a decision rather than a rewrite.
