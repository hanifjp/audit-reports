# RustFund Re-Audit — Lesson-Memory Feedback Loop in Practice

**Author:** Hafiz Labs — internal capability showcase
**Type:** Practice / training exercise (not an official contest submission)
**Target:** `CodeHawks-Contests/2025-03-rustfund` — a minimal Solana/Anchor crowdfunding program
**Date of exercise:** September 2026

---

> ### Disclaimer
> This is a **practice/training exercise**, not an official audit engagement or a contest
> submission. The target code is from the **CodeHawks "Rust Fund" First Flight**
> (`CodeHawks-Contests/2025-03-rustfund`), a public, beginner-oriented educational security
> contest that **concluded in 2025**. We did **not** participate in the live contest, made
> **no official submission**, and claim **no placement, ranking, or reward**. We use the code
> here purely as a fixed target to re-run our own review process against an earlier internal
> baseline, and to measure how the pipeline has changed. Nothing here should be read as an
> endorsement by, or affiliation with, the contest organizer or the original project author.

---

## Why this exercise

Our earlier practice runs (an AMM known-answer benchmark and the Anchor team's
`sealevel-attacks` corpus) each surfaced a specific gap, and each gap was turned into a durable
lesson in our lesson-memory system. Since then the pipeline gained three concrete improvements:

- a **relevance/instinct filter** that classifies the program and pulls only the lesson-memory
  and checklist material that actually applies, instead of flooding the analysis with unrelated
  context;
- a **more mature lesson-memory store**, seeded by the gaps found in prior runs;
- a **QA honesty fix** that reports "inconclusive" on token-exhaustion instead of emitting a
  false "QA PASS".

The natural question is: *does a matured pipeline actually produce a different, better review of
the same code than it did before?* RustFund is a good re-test target — it is a small, single-file
crowdfunding program with a known family of bugs, so we can compare a fresh run directly against
our own earlier baseline on the exact same source. This write-up is about the **delta**, not a
score.

## Methodology — the same three-layer pipeline

Findings flow through three independent stages, identical in shape to our other practice runs:

1. **Primary analysis.** A structured, checklist-driven large-language-model pass over the full
   in-scope source, producing typed findings with severity and location. For this run the
   relevance filter selected the crowdfunding/vault-style checklist material and the lessons
   relevant to lamport handling and state accounting.

2. **Execution-based validation.** Reported issues are handed to a validator that attempts to
   build a runnable proof-of-concept and mark each claim proven / unproven, rather than trusting
   the model's assertion.

   > **Important note for this target:** RustFund, as reviewed here, is a **single-file snippet
   > with no buildable workspace** — no `Cargo.toml`, no surrounding test harness, no deployable
   > entrypoint to drive on-chain. A compiled proof-of-concept is therefore not constructible, and
   > as expected the validation layer returned **"unable to prove" for every finding**. That is the
   > **correct outcome for an isolated single-file target and not a detection failure** — the
   > findings below are code-level certain from static analysis. (PoC-proving is exercised on our
   > other, runnable practice targets.)

3. **Second-opinion QA.** A separate reasoning pass re-examines the program with a
   state-machine / internal-consistency lens, explicitly tasked with finding high-value bugs the
   first pass missed rather than restating existing findings.

Each stage is monitored for truncated or incomplete model output. Every stage in this run
completed in full, so each result below is a genuine, conclusive outcome rather than an artifact
of a cut-off analysis.

## Results — an honest delta against our earlier baseline

Our earlier internal baseline on this code was **6 findings**: direct-lamport manipulation, a
checks-effects-interactions (CEI) ordering issue, a missing withdrawal precondition, a
contribution amount never incremented, an unsafe-timestamp `unwrap`, and a dead one-time deadline
guard.

The re-run **reproduced 5 of those 6**, surfaced **1 new higher-value finding** the earlier
baseline never reported, and **corrected the severities** the model originally over-stated. It
also **dropped one low-value finding** (the timestamp `unwrap`) — which we deliberately add back
below for full coverage.

| # | Finding | Realistic severity | Layer | vs. baseline |
|---|---------|--------------------|-------|--------------|
| 1 | **`withdraw()` missing precondition (rug vector)** — creator can pull all `amount_raised` at any time; no goal-reached or deadline-passed check, so funds can be withdrawn immediately after creation, stranding contributors | High | Primary | reproduced |
| 2 | **`contribute()` never sets `contribution.amount`** — `amount_raised` is incremented but the per-contributor record stays `0`, so `refund()` (which reads `contribution.amount`) always pays out `0` → contributors can never reclaim funds | High | Primary | reproduced |
| 3 | **`withdraw()` never zeroes `amount_raised` — replay / stale accounting** ⭐ **NEW** — payout is not consumed and no `withdrawn`/`closed` flag is set, so the withdrawal is replayable while the account still holds lamports (rent-exempt balance and/or later contributions), and the `amount_raised == sum of live contributions` invariant is broken; `refund()` likewise never decrements `amount_raised` | High | Second-opinion QA | **new this run** |
| 4 | **Direct lamport manipulation** — balances are moved via `try_borrow_mut_lamports()` instead of a `system_program::transfer` CPI; permitted on a program-owned PDA but bypasses rent-exemption safety and compounds with the CEI issue | Medium *(model tagged Critical)* | Primary | reproduced, **downgraded** |
| 5 | **CEI (checks-effects-interactions) violation** — lamports are moved before state is updated (`refund` zeroes `contribution.amount` after the transfer; `withdraw` moves funds before any success check) | Medium *(model tagged Critical)* | Primary | reproduced, **downgraded** |
| 6 | **Dead one-time deadline guard** — `dealine_set` (sic) is initialised `false` and checked in `set_deadline`, but no path ever sets it `true`, so the `DeadlineAlreadySet` guard is dead code and the creator can reset the deadline arbitrarily (e.g. to keep postponing the deadline-gated `refund`) | Medium | Second-opinion QA | reproduced |
| — | **Unsafe timestamp `unwrap`** — `Clock::get().unwrap()` / `try_into().unwrap()` can panic instead of returning a handled error | Low | *(baseline; not re-surfaced this run — see below)* | **dropped this run** |

**Net: 5 of 6 baseline findings reproduced, 1 new High-value finding added, 2 severities corrected
downward, 1 low-value finding not re-surfaced.** We deliberately do **not** claim "found every bug"
or "the new pipeline is strictly better on every axis" — the dropped `unwrap` finding is exactly
the kind of honest caveat below.

### The new finding — why it matters

The most interesting result is finding #3, which the earlier baseline did **not** report.
`withdraw()` transfers `amount_raised` lamports out but never zeroes `amount_raised` and sets no
"already withdrawn" flag. That leaves two problems the earlier run missed:

- **Replay:** if the transferred amount is smaller than the fund account's remaining lamport
  balance (its rent-exempt reserve, or lamports from later contributions), `withdraw()` can be
  called again and succeeds using those lamports.
- **Broken invariant:** `amount_raised` no longer equals the sum of live contributions, and
  `refund()` never decrements it either — so downstream accounting drifts.

This is a distinct, higher-value bug than the "missing precondition" (#1) or the "direct lamport"
mechanics (#4) that the baseline already had. It surfaced because the second-opinion layer applied
a state-consumption / invariant lens that the primary checklist pass framed only as a precondition
gap. That is exactly the redundancy-across-reasoning-styles the multi-layer design exists to
provide.

### The severity correction — honesty over headline numbers

The primary model originally tagged the two lamport/CEI findings (#4, #5) as **CVSS 9.8 /
Critical**. That is **not** realistic on Solana: there is no EVM-style reentrancy on these paths,
and direct-lamport / CEI-ordering issues on a program-owned PDA are **Medium**, not Critical, in
the Solana execution model. We downgraded them accordingly. Reporting the *realistic* severity —
even though it makes the headline look less dramatic — is the point of the exercise, not the
inflated model number.

## Limitations & honest disclosure

- **This is a self-comparison, not a published answer key.** The baseline is our own earlier
  review of the same code, so the delta measures *our pipeline's change over time*, not
  performance against an independent official finding set.
- **No proof-of-concepts were produced, by design.** The reviewed target is a single-file snippet
  with no buildable workspace, so the validation layer correctly returned "unable to prove" for
  every finding. Detection and code-level reasoning are the metrics here; PoC-proving is exercised
  on our runnable targets.
- **One low-value finding was not re-surfaced.** The unsafe-timestamp `unwrap` (Low/informational)
  that the earlier baseline reported did not appear in the new run's ranked output. We add it back
  above for coverage completeness and flag the non-reproduction honestly rather than quietly
  dropping it — a matured pipeline changing what it prioritises is not the same as that issue
  ceasing to exist.
- **A single re-run is a demonstration, not a guarantee.** Reproducing 5/6 and adding one new
  finding on one program shows the improved pipeline behaves differently and, here, better; it is
  not a statistical claim across all future crowdfunding code.

## Takeaway

Re-running a matured review pipeline over the same RustFund code produced a **materially different
and stronger result** than our earlier baseline: 5 of 6 prior findings reproduced, a **new
High-value bug** (withdrawal replay / `amount_raised` never zeroed) surfaced by the second-opinion
layer, and two over-stated Critical severities corrected down to a realistic Medium. We present
this as evidence of **continuous, honest improvement** — an audit process that gets sharper as its
lesson-memory and relevance filtering mature — rather than a claim of being perfect from the first
pass. The one non-reproduced low-value finding, and the "unable to prove" validation outcome on a
non-buildable single file, are disclosed plainly for the same reason.
