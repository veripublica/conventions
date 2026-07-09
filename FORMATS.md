# veripublica machine-output format

**Version 0.1.0.** The stable JSON a tool emits under `--format json`, so
one veripublica tool (or any external program — Sigil, Calibre, a CI job) can
consume another's output without bespoke parsing.

`human` format is for people and MAY change freely. **`json` is a contract:**
its shape is stable within a convention major version.

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
| `convention` | string | Convention major version this output follows (e.g. `"1.0"`). |
| `input` | string | The input path as given. |
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

## Guarantees

- Exactly one JSON object on stdout; nothing else on stdout in `json` mode.
- Stable within a convention major version: fields are not removed or
  repurposed; new optional fields MAY be added.
- The envelope is shared; the meaning of `items[].data` is each tool's own and
  is documented in that tool's docs.
