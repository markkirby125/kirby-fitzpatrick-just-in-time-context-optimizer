# Just In Time Context Optimizer — Technical Operational Dispatcher

**Framework Author**: William Fitzpatrick (*Writer Science*)  
**Source Lecture**: [The Top 1% of Experts Think on Paper—Here's How](https://www.youtube.com/watch?v=VkXMlrvq29o)  
**Parent Collection**: [Master Collection](../../kirby-fitzpatrick-writers-collection/SKILL.md) | [Global Help](../../../kirby-help/SKILL.md)  

---

## 1. Cognitive Foundation: The Workbench Is Not a Warehouse — Retrieval as Deliberate Thought

**The concept.** Fitzpatrick's diagnosis is an ordering error, not a discipline problem. Most people *plan the piece in their head, then write it out* — and the result comes out flat, because that workflow only ever **assembles what you already know**. The top 1% invert it: they **write to find out what they think**, then use a **reverse outline** to recover the structure they actually built, then **convert the private draft for a reader whose route was never theirs**. The machine can assemble; it cannot *have something to say*. Fitzpatrick's line is the whole thesis: *the thinking is the part that can't be automated.*

Now transposed to context engineering, verbatim in structure:

| Writer's move (Fitzpatrick) | Agentic retrieval equivalent |
|---|---|
| Plan in your head, then transcribe the plan | Decide the answer, then load every file that might be involved — ask the model to *assemble what you already assumed* |
| **Write to find out what you think** | **Retrieve to find out what the system does** — the read is the act of thinking, not a preparatory chore |
| A messy private draft, deliberately wrong first | A cheap, wrong, *bounded* probe: one search, one signature, one stack frame — before any expensive read |
| **Reverse outline** the draft to find the structure you built | Compress the probe results into a **symbol map** (file:line → role) before requesting a second read |
| Convert the draft for a reader whose route was never yours | Render only the excerpt the *next reasoning step* (or the reviewer, or the next agent) actually needs |
| Working memory holds a handful of items — keep **keywords, not sentences** | The context window holds a handful of *open questions* — keep **line ranges and signatures, not file bodies** |
| "Paper is the workbench of thought" | The context window is a **workbench**, sized for the work in hand — never an archive |

**The engineering inversion. And the two ways to fail it.** There are exactly two eager-retrieval workflows, and both skip the retrieval *decision*:

1. **"Read the whole file — I'll find it myself."** The model is handed 600 lines so that 9 of them can be found by attention. This is the machine doing the job it is *good* at (mechanically reproducing tokens) and *bad* at (attending across 40k tokens of irrelevant code). You have paid full price for the wrong skill.
2. **"The human names the file — the agent reads all of it."** Feels responsible, because a human made a choice. But *naming a file is not scoping a read*. You can be exactly right about which file matters and wrong about every line in it.

Both produce the same defect: **the retrieval route was never authored**, so the context contains facts nobody asked for — and irrelevant code is not neutral filler. It is an *anchor*: a plausible-looking sibling function (`invoice.ts:88`) sits in the window and outcompetes the correct one (`money.ts:141`) for attention. A missing fact yields a question. A **wrong fact in context yields a confident answer.** That asymmetry is why eager dumping is not merely expensive — it is *the* mechanism by which agents get things wrong while feeling informed.

**Why "just in time" and not "just in case."** Eager loading is a bet that cheap input beats cheap attention. It loses on five separate ledgers at once: token cost, latency, cache invalidation, attention dilution (lost-in-the-middle), and error amplification. JIT retrieval bets the other way — the *decision* about what to read is the scarce resource, and it is the one thing Fitzpatrick says cannot be automated. Which means: **an agent that dumps whole files is spending the automatable labor and skipping the non-automatable labor.** It has the economics exactly backwards.

**Priority, not economy.** JIT is not minimalism for its own sake. A 30-line range read for the *wrong* question is more expensive than a 300-line read for the right one. The protocol optimizes for **answer-bearing density per token**, and the metric is a ratio, not a cap: `citation yield = cited excerpts ÷ retrieved tokens`. Below ~0.5 in the final artifact, the retrieval was a dump wearing a range's clothing.

```text
BEFORE — EAGER DUMP  ("load everything, let attention sort it out")

  Q: "why does the invoice total drift by one cent?"
        │
        ▼   one request · six file bodies · ~48k tokens · 9 lines of signal
  ┌──────────────────────────────────────────────────────────────────────┐
  │ CONTEXT WINDOW — the workbench, used as a warehouse                  │
  │                                                                      │
  │  invoice.ts       612 L  ░░░░░░░░░░░░░░░░░░░░   2 relevant           │
  │  worker.ts        430 L  ░░░░░░░░░░░░░░░░░░░░   0 relevant           │
  │  money.ts         188 L  ░░░░░░░▒▒▒▒▓▓▓▒▒░▒░░   9 relevant ◄─ answer │
  │  invoice.test.ts  740 L  ░░░░░░░░░░░░░░░░░░░░   0 relevant           │
  │  types.d.ts        96 L  ░░░░░░░░░░░░░░░░░░░░   0 relevant           │
  │  package.json      58 L  ░░░░░░░░░░░░░░░░░░░░   0 relevant           │
  │                                                                      │
  │  money.ts:141 sits at 62% depth ── lost-in-the-middle, never cited   │
  └──────────────────────────────────────────────────────────────────────┘
        │
        ▼
  ✗ plausible answer assembled from invoice.ts:88 (a sibling function)
     turn 2: "no — money.ts"        → 3× the cost of the good run
     turn 3: still wrong, now with 3 stale files still resident as anchors
```

```text
AFTER — JIT RETRIEVAL  (the route is the artifact; the excerpt is the receipt)

  Q1 "which module rounds money?"            Q2 "what rule does it apply?"
        │                                          │
        ▼                                          ▼
  ┌──────────────────────────┐              ┌───────────────────────────┐
  │ probe: symbol index      │              │ read: money.ts L126–L149  │
  │ rg -n "round|toFixed"    │              │ 24 L · ~180 tok           │
  │ 3 hits · ~40 tok         │              │ ▓▓▓ the divergent rule ▓▓ │
  └──────────────────────────┘              └───────────────────────────┘
        │
        ▼
  ┌───────────────────────────────────────────────────────────────────────┐
  │ RETRIEVAL LEDGER                                                      │
  │  #  request                    tok    question it answered     kept?  │
  │  1  rg index (repo-wide)        40    where is rounding set?   map    │
  │  2  money.ts L126–L149         180    which rule applies?      yes    │
  │  3  money.ts L12–L24            90    what does helper expect? yes    │
  │  4  worker.ts  (whole file)  0→ABORT  question already closed  ✗ no   │
  │  ───────────────────────────────────────────────────────────────────  │
  │  retrieved 310 tok  ·  cited 231 tok  ·  citation yield 0.75          │
  │  answer cites money.ts:141 — at 4% of the window, not 62% depth       │
  └───────────────────────────────────────────────────────────────────────┘
```

**The mental model to hold.** *You are not reading code; you are spending attention.* Every request buys tokens and pays in signal; the protocol's job is to make each purchase name its question, its range, and its yield — and to forbid the one purchase that cannot be justified at all, the whole-file dump. The private dump is how you think; the route is what you show.

**Related dispatchers.** Navigation decides *what* to look at and JIT decides *how much of it to pull* — pair this with the [Codebase Navigation Router](../../kirby-fitzpatrick-codebase-navigation-router/SKILL.md). Freeze ground truth you must not mutate while probing with [Read-Only Vault Isolation](../../kirby-fitzpatrick-read-only-vault-isolation/SKILL.md). Name what a range read cannot resolve with the [Semantic Gap Hunter](../../kirby-fitzpatrick-semantic-gap-hunter/SKILL.md). Compress what you *do* keep with [Cargo-Weighted Syntax](../../kirby-fitzpatrick-cargo-weighted-syntax/SKILL.md) and the [Lexical Anti-Bloat Filter](../../kirby-fitzpatrick-lexical-anti-bloat-filter/SKILL.md); keep the objective ahead of the cosmetics with [Substance-First Refactoring](../../kirby-fitzpatrick-substance-first-refactoring/SKILL.md).

---

## 2. Core Transformation Protocols

### Rule 1 — The Default Retrieval Unit Is a Range or a Symbol, Never a File

A file body is not a retrieval; it is a hypothesis that the file is small enough to be harmless. Every read must name three things *before* it executes:

```
target   path  +  (line range | symbol name)  +  revision (sha / dirty marker)
question the single factual question this read answers, in one clause
budget   the token ceiling you will accept for that answer
```

A request missing any of the three is a dump. If you cannot write the question, you do not yet need the read — you need a cheaper probe (Rule 2).

### Rule 2 — Anchor First, Expand Second, Body Last

Escalation ladder — stop at the rung that answers the question:

| Rung | Request | Typical yield | Use when |
|---|---|---|---|
| 0 | Path/glob listing, symbol index, `documentSymbol` outline | 30–300 tok | you don't know the shape of the file |
| 1 | Search hits with **no** context (`rg -n`) | 20–200 tok | locating a name, call site, config key |
| 2 | Signature + type + first doc line (`LSP: documentSymbol` / `rg -n -A3 "^(export )?(async )?(function|class|const|def)"`) | 15–40 tok per symbol | you need the *contract* |
| 3 | Bounded hunk (`-C 3`, or an explicit range) | 60–400 tok | you need the *behavior* at one site |
| 4 | Whole function body | 200–4,000 tok | the logic itself is the question |
| 5 | Whole file | 4k–40k tok | **forbidden** without Rule 8's declared exception |

**Rung 5 is not a rung; it is a surrender.** If a question truly spans a file, it is almost always a question about *order invariants* or *cross-references* — answer it with an outline (Rung 0) plus two hunks, not a listing.

### Rule 3 — Compute the Budget From Open Questions, Not From Unfamiliarity

The refill trigger must be an **unresolved fact**, never a feeling.

```
budget = Σ over open questions (expected tokens to close it)
refill  ⇔  ∃ q : q still open  ∧  q is answerable by a named range
abort   ⇔  the next read answers no open question
```

"Let me also grab `worker.ts` to be safe" is not a refill; it is pre-emptive anchoring. **Unfamiliarity is not a question.** If nothing is open, the retrieval phase is over — go read your own ledger and answer.

### Rule 4 — Ask in Shapes, Not Sentences

Prefer the *contract* over the *implementation* wherever the question permits it: signature, types, exceptions, schema, topic, route, migration name, default value. A signature is ~15–40 tokens and closes a compatibility question that a 400-line body (~2,500 tokens) closes no better — while contributing 2,460 tokens of distractor. Bodies answer *how*; signatures answer *what* and *who*. Most engineering questions are *what* and *who*.

### Rule 5 — Every Request Carries Its Question; Every Answer Carries Its Citation

Keep a live ledger in the working context:

```
#  request                    tok   question answered             kept
1  rg -n "round|toFixed"      40    where is rounding defined?    map only
2  money.ts L126–L149        180    which rule applies?           yes
3  README.md                     0    ✗ no question open          aborted
```

Two invariants: a row with no question is deleted on sight, and an answer that cannot be cited as `path:line@sha` is recorded as **inference**, not evidence. The ledger also makes retrieval *idempotent*: never re-request a range you already hold — re-reading a file after an edit means re-reading the **one invalidated range**, not the file.

### Rule 6 — Retrieval Is Iterative: Probe → Reverse-Outline → Narrow

Fitzpatrick's loop ("make it wrong, make it shorter, make it again") applied to context:

1. **Make it wrong, cheaply.** First read is deliberately coarse and short: a search index, an outline, the throwing frames of a stack trace. Wrong is fine; *bounded* is mandatory.
2. **Make it shorter.** Reverse-outline the probe output into a symbol map — `file:line → symbol → role` — and discard the raw probe output. The map is the durable artifact; the search hits are scaffolding.
3. **Make it again, narrower.** Each subsequent request halves the range around the surviving hypothesis. Three narrowing passes without new information is a *signal-gap* finding, not a reason for a fourth: escalate to a named gap (Rule 8) instead of opening the file to look around.

### Rule 7 — Convert the Private Draft for the Reader Behind You

The dump is your private draft; it is never the deliverable. Render the **reader's route** explicitly: which ranges matter, in which order, and what each one proves. This holds for the human reviewing your answer, for the next agent in a pipeline, and for your own turn N+1 (whose working memory is not your turn N's). A route of three ranges plus a one-line "why" costs ~50 tokens and replaces a pasted file.

### Rule 8 — Declare Exceptions Instead of Falling Back to Dumping

Some facts are not in any range. Name them and pay deliberately:

| Exception | Why a range fails | Required compensation |
|---|---|---|
| Generated / vendored source | The authoring source changed, not this file | Read the generator input or schema; cite the generator, never the output |
| Cross-file invariant (ordering, idempotency, locking) | Distributed across ≥3 sites | 3 bounded hunks + an explicit invariant statement; if one site can't be quoted, it is `[UNMAPPED]` |
| Dynamic dispatch / reflection / string-keyed routing | No static edge exists | Enumerate the registry/config range; probe at runtime |
| A range read that genuinely fails | The question is "is this whole thing coherent?" | ≤1 named whole-file read, in the ledger, with its question and the token cost spelled out |

An undeclared whole-file read is the failure mode. A **declared** one, in the ledger with its question, is a budget decision.

### Rule 9 — Evict Resolved Ranges; Never Let a Dead Excerpt Stay Resident

A resolved hunk still in context is a distractor and an invitation to re-use a stale fact. On closure: replace the excerpt with its one-line conclusion plus the `path:line` pointer, and drop the body. Also evict *before* refilling — retrieval that only ever grows terminates in a dump regardless of how each request started.

### Anti-Pattern → Clean Replacement Ledger

| Anti-Pattern (Eager Dump) | Clean Replacement (JIT Retrieval) |
|---|---|
| `cat src/invoice.ts` / read the whole file "to be safe" | Rung 0 outline (`documentSymbol`), then the 9-line hunk that answers the question |
| "Paste the whole file and I'll find it myself" | "Here is the symbol map and 3 candidate ranges; request the one you need" |
| Re-reading a file after every edit | Read ledger + re-read only the invalidated range (edit ⊆ read, per range) |
| Pasting a 400-line stack trace | Keep the failing frames + the throwing symbol; prune the boilerplate chain |
| Retrieving `node_modules`, lockfiles, dist bundles, generated clients | Retrieve the source of truth (the schema, the template, the interface) and cite the generator |
| Summarizing a file you never opened | Read the signature first, summarize *from the contract*, and mark every inference |
| "Read the whole repo and tell me how X works" | Sequence from the anchor the error/question gives you; one rung at a time |
| `rg -A 500 "pattern"` to "get context" | `rg -n` index → targeted range read at the hit that matters |
| Preloading 20 files before any question exists | Ask the question first; preloading is assembling what you already assumed |
| Keeping every retrieved hunk resident "in case" | Evict on closure, keep `path:line`; the window is a workbench, not an archive |
| Re-requesting a symbol because you forgot reading it | Ledger makes retrieval idempotent: check first, then request |
| "I read the whole file, so I'm sure" | Certainty comes from a cited range, not from volume: `path:line@sha` or it is inference |

### The STOP Signals

```text
✗  a read with no named question or no token budget
✗  a whole-file read without a Rule 8 declaration in the ledger
✗  a second read of the same range in one session
✗  a retrieval that ignores its own previous hits (re-search, not refine)
✗  generated / vendored / lockfile content inside the window
✗  an answer citing no path:line, only "the code"
✗  "let me also check…" with no open question to close
✗  a fourth narrowing pass on the same hypothesis
✗  retrieved tokens ≫ cited tokens with no ledger explaining the residue
```

---

## 3. Engineering Application Scenarios

### 3.1 Code Reviews — Review the Retrieval Route, Not the File Dump

The reviewer's failure mode is the same as the author's, mirrored: a review packet that arrives as **three whole modified files** forces the reviewer to reconstruct a route that was never authored. Review the *route first*, the code second — and never accept a full-file paste as evidence of care.

```text
REVIEW ENTRY SEQUENCE
1. ROUTE    Which ranges did the change require? (signature deltas · changed hunks · call sites)
2. QUESTION What question did each read answer? Is any read unaccounted for?
3. ANCHORS  Which unrelated code is sitting in this packet, competing for attention?
4. EVIDENCE Does every claim cite path:line@sha, or is it inference dressed as reading?
5. Only now behavior · tests · naming · style
```

A review packet assembled the JIT way:

```text
DIFF (3 hunks · 41 changed lines)
  money.ts L141–L149        roundHalfUp → roundHalfEven
  money.ts L12–L24          helper doc + default arg
  invoice.ts L88–L96        call site (unchanged behavior, now explicit arg)

RANGES READ (ledger)
  money.ts L120–L160   (41 L)  which rule applies to line totals?
  invoice.ts L80–L100  (21 L)  does the caller depend on half-up at the boundary?
  rg -n "roundHalfUp"  (7 hits) who else calls the old helper?
  → 1 caller in the test fixture · 0 in services · declared residue: fixtures/ (not read)

NOT READ, ON PURPOSE
  worker.ts, invoice.test.ts  → no call path touches rounding (index shows 0 hits)
```

| Dump-Review Comment | JIT-Review Comment |
|---|---|
| *"LGTM — read the whole diff, looks fine."* | *"Show the call-site range for `roundHalfUp`; the grep index lists 7 hits and the packet only accounts for 2."* |
| *"Paste the full file so I can check."* | *"Paste the outline plus L120–L160; I don't need the other 450 lines to answer this."* |
| *"Is this change safe?"* | *"This hunk changes a boundary rule. Which range did you read to establish that no caller relies on half-up? Cite it."* |
| *"There's a lot going on here."* | *"Three hunks, one question each. Hunk 2 and hunk 3 don't share a question — is this one change or two?"* |

**The blocking verdict, stated plainly:**

> "I'm blocking on retrieval discipline, not on style. This packet is three whole files (~2,100 lines) for a 41-line change: the reviewer is being asked to *find* the change before evaluating it, which means the anchoring is mine to do rather than yours. Send the changed hunks with `path:line@sha`, the call-site range for the changed symbol, and the grep index that justified *not* reading `worker.ts` and `invoice.test.ts`. That's a 60-line packet and a one-pass review — the current one is a 2,100-line packet and a three-pass review that will still miss the fixture caller."

### 3.2 PR Descriptions — The Retrieval Receipt as the Spine

A PR body is not a summary of the diff; it is the **route** that explains why the diff is the right size, plus the receipt for what was *not* read. Omit the receipt and the reviewer re-walks your retrieval in the most expensive possible way: by reading everything.

```markdown
## What changed
`Invoice.currency`: absent → required ISO-4217 string

## Retrieval receipt
| # | request                      | tok | question answered              | kept |
|---|------------------------------|-----|--------------------------------|------|
| 1 | rg -n '"invoice"' -g '!dist' |  60 | who parses the payload?        | map  |
| 2 | invoice.ts L118–L176         | 210 | how is the payload built?      | yes  |
| 3 | tax_client.py L40–L72        | 140 | does the consumer read strict? | yes  |
| 4 | schemas/invoice.v1.json      |  90 | is the field optional there?   | yes  |
| — | retrieved 500 tok · cited 380 tok · yield 0.76                            |

## Closed by retrieval
- `tax_client.py:58` parses with `model_validate` → strict → **required field is breaking**
- `schemas/invoice.v1.json:14` has `additionalProperties: false` → field must be registered

## Not read, and why (declared residue)
- `worker.ts` — index shows 0 references to the payload key
- `mobile/` — vendored client; tracked as [UNMAPPED], probe scheduled in staging

## Reviewer entry point (start here, in this order)
1. `invoice.ts L118–L176` — the shape change, 58 lines
2. `tax_client.py L40–L72` — the consumer that makes it breaking, 32 lines
3. `schemas/invoice.v1.json L10–L20` — the registration that fixes it, 10 lines
   ≈ 100 lines total; the rest of the diff is mechanical.
```

Two properties make this body load-bearing: **the residue is named** (so the reviewer knows the boundary of the claim instead of assuming completeness), and **the reader's route is written down** — Fitzpatrick's "convert the private draft for a reader whose route was never yours," applied to a diff. A PR that pastes whole files is a private draft published by accident.

### 3.3 Architecture RFCs / ADRs — Think on Paper, Publish the Map

An RFC built by eager dumping becomes an appendix nobody reads and a body nobody can verify. Keep the **discovery output** out of the decision and the **evidence excerpts** inside it:

1. **Evidence is excerpts, never listings.** Every claim carries a signature, a schema fragment, or a bounded hunk — `path:line@sha`, always. Whole-module appendices move behind a link; a reviewer who needs the listing can request the range.
2. **Reverse-outline the discovery into the decision structure.** The messy probe log (searches, dead ends, wrong hypotheses) is not the RFC. It compresses into: constraints → options → decision → consequences. The raw log survives as an optional appendix marked *"discovery log — not normative."*
3. **State what no range could settle.** Cross-file invariants, dynamic dispatch, and behavior that only exists in production go into a `[UNMAPPED]` section with the probe that would close each one — this is the ADR's honest residue, and its absence is the tell of a dumped RFC.
4. **Fix the revision in the citation.** `path:line` drifts. `path:line@sha` is a fact; a floating line number is a rumor that will be quoted for a year.

| ADR as Dump | ADR as Retrieval Route |
|---|---|
| "Here are the four modules involved." (4 full listings) | "Four excerpts, 62 lines total: the two signatures that conflict and the two call sites that share a transaction." |
| "The performance profile shows a bottleneck." | "`metrics.go:88` (the histogram) and `handler.go:214` (the hot path) — 24 lines; the p99 gap is in hop 2, not hop 1." |
| "We reviewed the schema and it looks extensible." | "`invoice.v1.json:14` sets `additionalProperties:false`; the claim 'extensible' is an inference until that line changes." |
| "Nothing else depends on this symbol." | "`rg -n` index: 7 hits, 2 in-repo callers, 0 in services; the vendored client is `[UNMAPPED]`." |

The last row is where this pays: an untraced RFC cannot be *checked*, so it will be re-litigated at the incident, when the cost of re-deriving the route is highest and the reviewer is least patient.

---

## 4. Verification Checklist

- [ ] **No undeclared whole-file reads.** Every retrieval in the session names a range or a symbol; every whole-file read (if any) appears in the ledger with its question, its token cost, and its Rule 8 justification. I did not reach zero by not looking — the count is spelled out.
- [ ] **Every request carried a question and a budget before it executed.** The ledger shows `# / request / tokens / question answered / kept`; rows with no question were aborted rather than filled, and `retrieved tok` is compared against `cited tok` with a yield I am willing to state out loud.
- [ ] **The route ascends the ladder and stops at the answer.** Each file's first touch was Rung 0–2 (index, outline, signature), bodies were reached only when behavior was the question, and no hypothesis got a fourth narrowing pass without being escalated as a named gap.
- [ ] **Resolved ranges were evicted and the map was kept.** Duplicate reads of the same range: zero. Each closed question left a one-line conclusion with its `path:line`, not the hunk; each edit invalidated exactly the ranges it touched, and only those were re-read.
- [ ] **Every claim in the deliverable is cited `path:line@sha` or explicitly marked as inference.** Unsupported facts, generated/vendored content, dynamic-dispatch edges, and cross-file invariants that no range could settle are listed as `[UNMAPPED]` with an owner and the probe that closes them — and I can show the retrieval route to anyone who asks, in the order they should read it.