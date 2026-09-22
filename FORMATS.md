# veripublica machine-output format

**Version 0.6.0.**

> **Implemented.** epubveri **v0.5.0** (2026-07-11) shipped the first
> `--format json`; epubsana **v0.2.0** followed. The envelope is an **observed
> contract**: from those releases on, its shape is stable within the
> convention's stability boundary ([CLI.md §9](./CLI.md#9-versioning)) — the
> guarantee this document carried as a promise while it was provisional
> (v0.1.0–v0.4.0).
>
> The `json` format name is reserved by
> [CLI.md §3](./CLI.md#3-options) for every tool: one that has not implemented
> it **rejects** `--format json` rather than falling back to `human`.

This document specifies the JSON a tool emits under `--format json`, so one
veripublica tool (or any external program — Sigil, Calibre, a CI job) can consume
another's output without bespoke parsing.

`human` format is for people and MAY change freely. `json` is a contract: its
shape is stable within the convention's stability boundary
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
  "tool_version": "0.17.0",
  "convention": "0.6",
  "status": "error",
  "dry_run": false,
  "inputs": [
    {
      "path": "a.epub",
      "status": "ok",
      "summary": { "fatals": 0, "errors": 0, "warnings": 0, "infos": 0, "usages": 0 },
      "items": []
    },
    {
      "path": "b.epub",
      "status": "problems",
      "summary": { "fatals": 0, "errors": 3, "warnings": 1, "infos": 0, "usages": 0 },
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
`a.epub` and `b.epub`: the process-every-input rule, in JSON form. Note
`c.epub`: no verdict was possible, so it carries **no** `summary` at all — its
`status` and `error` are the whole answer, and there are no counters to report
([§1.4](#14-counters-and-what-a-filtered-run-must-record)).

A **transformer**'s input object additionally reports **what it wrote** — the
same fact its human output already prints (`wrote book_fixed.epub`):

```json
{
  "path": "book.epub",
  "status": "ok",
  "output": "book_fixed.epub",
  "summary": {
    "fatals_before": 0, "errors_before": 150, "warnings_before": 12,
    "infos_before": 0, "usages_before": 4,
    "fatals_after": 0, "errors_after": 0, "warnings_after": 12,
    "infos_after": 0, "usages_after": 4,
    "applied": 2, "skipped": 0, "proposed": 0
  },
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
| `convention` | string | The convention's **stability key**: the version prefix at which stability is guaranteed — `"0.6"` while the convention is `0.x`, `"1"` from `1.0.0` on ([CLI.md §9](./CLI.md#9-versioning)). Compare with string equality; there is nothing finer to parse. The key is **asserted by the emitting tool about itself**: a shared implementation takes it from the tool rather than stamping its own, so that a tool never claims a version it has not implemented by inheriting one from a dependency. |
| `status` | string | Mirror of the exit code, aggregated over the inputs: `"ok"` (every input clean / every goal met) → `0`; `"problems"` (every input processed; error- or fatal-severity findings, or an unmet goal, remain — [CLI.md §6](./CLI.md#6-exit-codes)'s threshold) → `1`; `"error"` (**at least one input could not be processed**) → `2`. A tool that could not run at all emits no envelope. |
| `dry_run` | boolean | `true` when the run was `--dry-run`: the identical shape, items describing what *would* be done (every item's `outcome` is `"proposed"`), `output` naming what *would* be written. Absent means `false`. The flag is a summary of the items; the two can never disagree. |
| `summary` | object | Optional aggregate counts (small, flat, tool-specific). Derivable from the inputs; a consumer MUST NOT require it. What its counters must report, and what a filtered run must record in it, are in [§1.4](#14-counters-and-what-a-filtered-run-must-record). |
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
| `summary` | object | Tool-specific counts for this input (small, flat). Optional — see [§1.4](#14-counters-and-what-a-filtered-run-must-record). |
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
| `outcome` | string | What happened to the item: `"applied"`, `"skipped"`, `"proposed"`, or `"reverted"`. **Required** on `"fix"` and `"operation"` items; never present on `"finding"` items. See below. |
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
([CLI.md §6](./CLI.md#6-exit-codes)). An invocation MAY withhold items of a
severity from the report; when it does, it withholds them from **every** format
the run can emit, and the envelope records it — see
[§1.4](#14-counters-and-what-a-filtered-run-must-record).

**`outcome`** carries *"did, or would?"* per item, because a consent-per-fix
repairer routinely mixes both in one ordinary run. `"applied"` — the change
was made. `"skipped"` — presented and not done: the caller declined.
`"proposed"` — no decision exists yet: a dry run, or a consent that arrives
after the report. `"reverted"` — the tool applied this fix and then undid it,
because applying it produced a defect that was not present before. The fix is
**not** in the output and the caller did **not** decline it; the finding it
addressed is unrepaired. Under `dry_run: true` every item is `"proposed"`. A
transformer that always applies everything stamps `"applied"` on every item —
the field being **required** on `fix`/`operation` items is what keeps its
absence from meaning anything. The value set is closed, like `severity`'s.

Two boundaries hold `"reverted"` to that one meaning:

- It means **undone after re-validation**, not *"failed to apply"*. A change
  the tool could not make — an I/O failure mid-run — is an exit-`2` condition
  ([CLI.md §6](./CLI.md#6-exit-codes)), never an item outcome.
- It is **one value, not a family.** Why a fix was reverted is the finding it
  produced, in the detector's own vocabulary; it belongs in tool-owned `data`,
  not in a closed set of revert reasons.

### 1.4 Counters, and what a filtered run must record

`summary`'s keys are each tool's own vocabulary ([§2](#2-what-is-standard-and-what-is-each-tools)).
Two rules bind it anyway, and both are **conditional**: they reach a tool only
where it has already chosen to do the thing they describe.

**A counter over a closed set reports every member.** A counter keyed by a
closed value set **this specification declares** — `severity`, `outcome` —
reports **every member the tool has a concept of**, including zero, and the set
does not vary from run to run.

- *Has a concept of* is [§1.3](#13-item-fields)'s existing test, not a new one: a
  non-EPUB tool that never has a `usage` finding counts no usages, and a
  repairer whose build cannot revert reports no `reverted`. It does **not** mean
  *whatever the tool's summary happens to list today* — that reading would let a
  tool decline every member by declaring none.
- It is not one-dimensional. A tool that counts a severity in two tenses
  (`errors_before`, `errors_after`) reports every member it has a concept of in
  each tense.
- This does not contradict *"fields that don't apply MAY be omitted"*: that
  permission is about fields that do not apply, and a counter over a set the
  tool observes always applies. Zero is an answer; absence is not.

**A filtered run records that it was filtered.** Where an invocation withholds
items from the report, the summary object carrying the affected counters names
those severities in **`suppressed`**:

```json
"summary": { "fatals": 0, "errors": 2, "warnings": 1, "infos": 0, "usages": 0,
             "suppressed": ["usage"] }
```

The counter keys above are one tool's own spelling — `summary`'s vocabulary is
tool-owned ([§2](#2-what-is-standard-and-what-is-each-tools)), and another tool's
may be singular, or tensed. `suppressed`'s **values** are not: they are severity
names from [§1.3](#13-item-fields)'s set.

- `suppressed` records **the gate**: the severities for which a format-level
  filter was in effect, whether or not it removed anything on this run. Absent
  or empty means **no format-level filter was in effect** — every item the run
  produced is present. It is a completeness marker, not a "something is hidden"
  flag: it answers *can I trust this counter?*
- A severity named there may be **incompletely represented** in `items` and in
  the counters: the format holds some or none of what the run produced at that
  severity.
- It lives in the **same object as the counters it qualifies**, per-input
  summaries included. A qualifier that a consumer can read without reading the
  claim it qualifies is not a qualifier.
- **No count of what was withheld is required.** A tool that filters at source
  cannot count what it never produced, and nothing in this specification obliges
  it to.
- `suppressed` is a **reserved non-counter member** of the summary object.
  Summing a summary's values was never safe — a tool's own vocabulary may hold
  tenses, outcomes and strings in one object — and this makes it plainly unsafe.

**Omission may not dodge the marker.** A run that applies a format-level filter
**MUST NOT** omit a summary object it would otherwise emit. For an input that
**produced a report** while a filter was in effect the marker MUST be present —
a tool that emits no summary for such an input emits one carrying `suppressed`
alone, which is conformant because the counter rule above is conditional and
that object has no counter to complete. An input that produced **no** report —
no verdict was possible — carries no summary and no marker: its `status` already
says the counters do not exist, and a missing summary makes no claim about
filtering in either direction. *Absent or empty means no filter* is about
`suppressed` **within** a summary object, never about a summary that is not
there.

**What counts as suppression.** The same run's own report holds the item and a
format does not. A run that produced fewer items because it was **asked a
narrower question** — a flag that selects which checks run, a profile, a version
— has withheld nothing and records nothing.

**Two limits.**

- Suppression may hide what was **found**; never what was **done**. An item
  recording a change made to the user's file is never withheld. A verifier
  hiding a finding hides information about the book; a repairer hiding an
  applied `fix` would hide what the tool did to the user's file, which is the one
  thing a change report exists to state.
- These rules govern an **envelope**, not an API. Nothing here reaches a
  library, and that silence is deliberate rather than an omission: a filtered
  CLI report is recoverable by running again, while a library that answered a
  question with part of the answer would be unrecoverable by its caller. This
  document does not legislate there.

## 2. What is standard, and what is each tool's

**The skeleton is standard**: the envelope fields, the input object, and the
shared item fields above. Without them a consumer can rely on nothing.

**The flesh is tool-owned**: `summary`'s keys and `items[].data`'s contents are
each tool's own vocabulary, documented in that tool's docs. Tool-owned is not
unruled: where a tool keys a counter by a closed set this document declares,
[§1.4](#14-counters-and-what-a-filtered-run-must-record) binds what that counter
must report — the rules describe the vocabulary a tool has already chosen, they
do not prescribe one.

The bridge between the two is the same rule the CLI uses for option names
([CONTRIBUTING §2](./CONTRIBUTING.md#2-the-governing-principle)): a `data` key
is born in **one** tool; when a **second** tool needs the same key, it is
promoted to a shared item field — by an issue on this repository, not by drift.

A **reference implementation** of the skeleton exists — non-normative: the
JSON above is the contract, the types are a convenience. It lives in
[`epubveri::envelope`](https://github.com/veripublica/epubveri) (the reference
tool, `epubveri = "0.13"` at the time of writing), generic over the two
tool-owned slots (`summary`, `data`), and is used by epubveri and epubsana. It
stays there until a veripublica **Rust** tool that does **not** depend on
epubveri needs the envelope, at which point it moves to its own crate — the
promotion rule above, applied to implementation shapes
([#27](https://github.com/veripublica/conventions/issues/27)). Because the
convenience and the contract can drift, a release that touches this document
carries *"update `epubveri::envelope` or invoke the promotion trigger"* in
epubveri's tracking issue.

## 3. Guarantees

- Exactly one JSON object on stdout; nothing else on stdout in `json` mode. The
  output is never colorized ([CLI.md §5](./CLI.md#5-streams-prompts-and-color)).
- `inputs` preserves command-line order.
- Stable within the convention's stability boundary: fields are not removed or
  repurposed; new optional fields MAY be added.
- The envelope's skeleton is shared; `summary` and `items[].data` are each
  tool's own and documented in that tool's docs.

All of the above binds since epubveri v0.5.0 (2026-07-11), the first
implementation — the moment the provisional period was defined to end. The
document spent four versions provisional, in contract language, precisely so
that hardening, when it came, was a decision rather than a rewrite.
