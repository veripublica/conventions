# veripublica machine-output format

**Version 0.4.0 — PROVISIONAL.**

> **No tool emits this envelope yet.** This document is a design proposal, not an
> observed contract: its shape MAY change without a major (or, while `0.x`,
> minor) version bump **until the first tool ships it** — the stability
> guarantee below begins at that first implementation, not before.
>
> The shape was redesigned in
> [#21](https://github.com/veripublica/conventions/issues/21) (multi-input,
> dry runs, a transformer's written output) precisely so the first
> implementation — epubveri's — does not harden an envelope that cannot
> describe its own tool's invocations.
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

## 1. The envelope

Every `--format json` invocation that **runs** prints exactly **one** JSON object
to stdout. A **usage error produces no envelope**: per
[CLI.md §5](./CLI.md#5-streams-prompts-and-color) it is short stderr text with
exit `2`, and stdout stays empty. The envelope describes runs that happened;
inputs that could not be processed are described *inside* it.

```json
{
  "tool": "epubveri",
  "tool_version": "0.5.0",
  "convention": "0.4",
  "status": "error",
  "dry_run": false,
  "inputs": [
    {
      "path": "a.epub",
      "status": "ok",
      "summary": { "errors": 0, "warnings": 0 },
      "items": []
    },
    {
      "path": "b.epub",
      "status": "problems",
      "summary": { "errors": 3, "warnings": 1 },
      "items": [
        {
          "type": "finding",
          "code": "RSC-005",
          "severity": "error",
          "location": "OEBPS/ch1.xhtml",
          "position": { "line": 44, "column": 9 },
          "message": "Error while parsing file: element \"img\" missing required attribute \"src\""
        }
      ]
    },
    {
      "path": "c.epub",
      "status": "error",
      "error": "cannot read: not a ZIP archive",
      "items": []
    }
  ]
}
```

This run exits `2` (at least one input could not be processed,
[CLI.md §6](./CLI.md#6-exit-codes)) — and still carries the full reports for
`a.epub` and `b.epub`: the process-every-input rule, in JSON form.

A **transformer**'s input object additionally reports **what it wrote** — the
same fact its human output already prints (`wrote book_fixed.epub`):

```json
{
  "path": "book.epub",
  "status": "ok",
  "output": "book_fixed.epub",
  "summary": { "errors_before": 150, "errors_after": 0, "applied": 2, "skipped": 0 },
  "items": [
    {
      "type": "fix",
      "outcome": "applied",
      "code": "RSC-016",
      "rule": "htm.entity.undeclared",
      "severity": "error",
      "location": "OEBPS/ch1.xhtml",
      "message": "Mapped 774 undeclared HTML entities to characters",
      "data": { "occurrences": 774 }
    }
  ]
}
```

### 1.1 Envelope fields

| Field | Type | Meaning |
| --- | --- | --- |
| `tool` | string | The tool's name (e.g. `"epubveri"`). |
| `tool_version` | string | The tool's SemVer. |
| `convention` | string | The convention's **stability key**: the version prefix at which stability is guaranteed — `"0.4"` while the convention is `0.x`, `"1"` from `1.0.0` on ([CLI.md §9](./CLI.md#9-versioning)). Compare with string equality; there is nothing finer to parse. |
| `status` | string | Mirror of the exit code, aggregated over the inputs: `"ok"` (every input clean / every goal met) → `0`; `"problems"` (every input processed; error- or fatal-severity findings, or an unmet goal, remain — [CLI.md §6](./CLI.md#6-exit-codes)'s threshold) → `1`; `"error"` (**at least one input could not be processed**) → `2`. A tool that could not run at all emits no envelope. |
| `dry_run` | boolean | `true` when the run was `--dry-run`: the identical shape, items describing what *would* be done (every item's `outcome` is `"proposed"`), `output` naming what *would* be written. Absent means `false`. The flag is a summary of the items; the two can never disagree. |
| `summary` | object | Optional aggregate counts (small, flat, tool-specific). Derivable from the inputs; a consumer MUST NOT require it. |
| `inputs` | array | One **input object** per `-i`, **in command-line order** — an array even when there is exactly one. |

### 1.2 Input object fields

Each element of `inputs` is self-contained — deliberately, so that other
surfaces of the same engine (a wasm binding's return value, for instance) can
reuse the shape as-is.

| Field | Type | Meaning |
| --- | --- | --- |
| `path` | string | The input path as given on the command line. |
| `status` | string | `"ok"`, `"problems"`, or `"error"` — this input alone. |
| `error` | string | Present only with `status: "error"`: why this input could not be processed. (Named `error`, not `message` — `message` already means something at the item layer.) |
| `output` | string | Transformers only: the path written — or, under `dry_run`, the path that would be written. The report states the facts of the run; what a consumer does with the path is not this specification's business. |
| `summary` | object | Tool-specific counts for this input (small, flat). Optional. |
| `items` | array | The findings / fixes / operations for this input. May be empty. |

`status: "error"` means **no report was possible** — it never grades a verdict.
A defect the tool can still name with a code, however severe (a
`fatal`-severity finding included), is a *verdict*: the input is `"problems"`,
and the finding is in `items`, where a consumer — a repairer above all — can
plan against it.

### 1.3 Item fields

Each item is an object. These fields are **shared** — a consumer can rely on
them across tools — and a tool MAY add more under `data`:

| Field | Type | Meaning |
| --- | --- | --- |
| `type` | string | Item kind: `"finding"` (verifier), `"fix"` (repairer), `"operation"` (transformer). |
| `outcome` | string | What happened to the item: `"applied"`, `"skipped"`, or `"proposed"`. **Required** on `"fix"` and `"operation"` items; never present on `"finding"` items. See below. |
| `code` | string | The stable, tool-facing code — for EPUB tools, the epubcheck-compatible message ID (e.g. `"RSC-005"`). |
| `rule` | string | Optional finer sub-code (e.g. `"ncx.ids.invalid_ncname"`). |
| `severity` | string | One of `"fatal"`, `"error"`, `"warning"`, `"info"`, `"usage"` — epubcheck's vocabulary, completing the `code` contract. See below. |
| `location` | string | Container-relative path the item concerns, if any. |
| `position` | object | `{ "line": N, "column": N }` (1-indexed), if known. |
| `message` | string | Human-readable one-liner. |
| `data` | object | Tool-specific extra fields (counts, before/after values, …). |

Fields that don't apply MAY be omitted. Consumers MUST ignore unknown fields
(so tools can extend `data` without breaking anyone).

**`severity`** is a reserved value **set**, not a requirement: a tool emits
only the values it has a concept of — a non-EPUB tool that never has a `usage`
finding simply never emits one. The set is **closed**: a new value enters by
issue and release
([CONTRIBUTING §2](./CONTRIBUTING.md#2-the-governing-principle): *no value is
invented for an unnamed need*), never through `data`. On a `fix` or
`operation` item, `severity` is **inherited** — the severity of the finding
the item addresses, verbatim from the detector — never how noteworthy the
report line is. Severities below `error` never move the exit code
([CLI.md §6](./CLI.md#6-exit-codes)). `usage`-severity items are always
present in `json`: the envelope is for machines, which filter;
suppression-by-default is a `human`-surface concern.

**`outcome`** carries *"did, or would?"* per item, because a consent-per-fix
repairer routinely mixes both in one ordinary run. `"applied"` — the change
was made. `"skipped"` — presented and not done: the caller declined.
`"proposed"` — no decision exists yet: a dry run, or a consent that arrives
after the report. Under `dry_run: true` every item is `"proposed"`. A
transformer that always applies everything stamps `"applied"` on every item —
the field being **required** on `fix`/`operation` items is what keeps its
absence from meaning anything. The value set is closed, like `severity`'s.

## 2. What is standard, and what is each tool's

**The skeleton is standard**: the envelope fields, the input object, and the
shared item fields above. Without them a consumer can rely on nothing.

**The flesh is tool-owned**: `summary`'s keys and `items[].data`'s contents are
each tool's own vocabulary, documented in that tool's docs.

The bridge between the two is the same rule the CLI uses for option names
([CONTRIBUTING §2](./CONTRIBUTING.md#2-the-governing-principle)): a `data` key
is born in **one** tool; when a **second** tool needs the same key, it is
promoted to a shared item field — by an issue on this repository, not by drift.

## 3. Guarantees — from the first implementation on

- Exactly one JSON object on stdout; nothing else on stdout in `json` mode. The
  output is never colorized ([CLI.md §5](./CLI.md#5-streams-prompts-and-color)).
- `inputs` preserves command-line order.
- Stable within the convention's stability boundary: fields are not removed or
  repurposed; new optional fields MAY be added.
- The envelope's skeleton is shared; `summary` and `items[].data` are each
  tool's own and documented in that tool's docs.

Until the first implementation ships, none of the above binds anyone — that is
what **provisional** means. It is written in contract language so that adopting
it, when the time comes, is a decision rather than a rewrite.
