# Contributing to veripublica conventions

This repository holds the shared contract the veripublica tools follow. It is a
**specification**, not code, and it changes slowly and on purpose.

The working language here is **English**.

---

## 1. What belongs in this repository

A rule belongs here only if the answer to this question is *yes*:

> **Without this rule, would two veripublica tools be incompatible with each
> other, or would a user who knows one tool be surprised by the next?**

If the answer is no, the decision belongs to the tool, not to the convention.

CLI.md already says it: *"Tools MAY add their own options and subcommands
freely."* `--goal`, `--auto-safe`, `--profile` are each one tool's business and
never come here.

### When a tool must open an issue here

Exactly three cases:

1. It wants to use a **reserved** name with a meaning other than the one defined.
   (It may not — but it may propose changing the definition.)
2. It wants a name **reserved**, so that the flag means the same thing in every
   tool.
3. It cannot satisfy a **MUST**, and needs the rule relaxed or an exemption.

Everything else: just build it.

## 2. The governing principle

**The convention codifies practice; it does not invent it.**

An option is born in **one** tool. When a **second** tool needs the same thing,
that is when it is reserved and written down here. This keeps the specification
from filling up with elegant ideas that nobody implements.

There is one exception, and it must be named as such:

- **Safety rules may precede practice.** A rule that prevents data loss (for
  example: never silently overwrite an existing file) does not have to wait for a
  second tool to ask for it.

Anything else invented here — before any tool implements it — is **provisional**
and must say so in its own document.

A second principle governs how alternatives are weighed once a rule *has* earned
its place:

**Rules are chosen on merit. The specification leads; the tools follow.**

What a tool currently does is evidence of what is broken, and an input to
migration sequencing — never the argument for what the rule should be. A past
mistake in a released tool (a misused reserved name, a silent fallback) is a cost
to schedule, not a constraint to design around: the tools are updated one at a
time, release by release, and the order is free to loop back when a migration
teaches us something.

## 3. How a change happens

**1. Propose.** Open an issue. State what breaks without the rule, not just what
would be nicer with it. Say which tools it affects and whether adopting it is a
behaviour change for any released tool.

**2. Discuss.** In the issue. Other veripublica projects weigh in there.

**3. Decide.** A **Decision** comment records what was decided, **why**, and what
was rejected along the way.

- `declined` issues are **closed** at this point. Never deleted: *"why we didn't
  do that"* is worth as much as the rules we did adopt.
- `accepted` issues stay **open**, and are closed by the pull request that writes
  them into the documents (step 4). The open `accepted` issues are therefore the
  queue of decisions taken but not yet written down — which is exactly what
  batching requires you to be able to see.

**4. Ship.** Accepted decisions are **batched**, not applied one at a time. A
single pull request writes the accumulated decisions into the documents, closes
the issues (`Closes #N`), updates `CHANGELOG.md`, and is released with a tag.

`CHANGELOG.md` is written **at this point, not at step 3**. The `accepted` label
is the only queue; a parallel list in `Unreleased` would be a second copy of the
same state, and the two would drift. (They did, within a day, the first time this
was tried.)

Batching is deliberate. If every decision landed on `main` the moment it was
made, the tools would be chasing a moving target — conforming one day and not the
next. Tools track **releases**, not `main`.

### Labels

| Label | Meaning |
| --- | --- |
| `proposed` | Open, under discussion. |
| `accepted` | Decided; open until a release writes it into the documents. |
| `declined` | Decided against, and closed. The issue records why. |

## 4. Versioning

The convention is versioned with [SemVer](https://semver.org/), with three
components — `0.1.0`, not `0.1`.

**Today the convention is `0.x`.** Per SemVer, *"anything MAY change at any
time"*: while we are below `1.0.0`, a rule can be added, changed, or removed
without ceremony. This is the cheapest moment to settle the contract, and it is
why we are settling it now, before the tools reach their own `1.0.0`.

Once `1.0.0` is released:

| Bump | Meaning |
| --- | --- |
| **major** | Anything that makes a conforming tool non-conforming. **Adding a MUST is a major change.** |
| **minor** | Additive normative change: a new SHOULD or MAY, a newly reserved name. |
| **patch** | **Strictly non-normative**: typos, examples, clarifications. |

The discipline that makes `patch` safe: if a "clarification" changes what a
conforming tool must do, it is **not** a patch, however small it looks.

### The stability boundary

A consumer needs to know which part of the version it can rely on. That boundary
moves once:

- while `0.x`: stability is per **minor** (`0.1` → `0.2` may break anything);
- from `1.0.0`: stability is per **major**.

Any version string a tool reports — in `--help`, in a document, in machine output
— carries the version **at that precision**: `0.1` today, `1` after `1.0.0`.

### Declaring conformance

A tool states the convention version it targets, e.g. *"conforms to veripublica
conventions v0.1"*. It must name a version that has been **tagged**. A claim
against `main`, or against an untagged `v1`, points at a moving document and
means nothing.

## 5. How the tools keep up

**They do not poll this repository.** Periodic checking does not happen; it is
forgotten, and returns six months later as a pile of debt.

Instead, when a release is tagged here, **this repository opens a tracking issue
in each affected tool's repository**: what changed, and what it means for that
tool specifically.

We never push code into a tool's repository. Deciding *how* to adopt a rule is
the tool's own work. Telling it *what changed and why it matters* is ours.

For tools that are already released, alignment must be **backward-compatible**:
add an alias, add a format, do not break an invocation that works today.

## 6. Licensing

The convention is licensed **[CC-BY-4.0](./LICENSE)**, deliberately permissive:
the point of a convention is to be adopted, including outside veripublica. By
contributing, you agree your contribution is licensed the same way.

This is the **specification's** license. It has nothing to do with the tools'
code, which is licensed separately.
