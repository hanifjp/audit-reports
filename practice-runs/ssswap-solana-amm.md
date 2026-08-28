# SSSwap (Solana AMM) — Multi-Layer Audit Pipeline Practice Run

**Author:** Hafiz Labs — internal capability showcase
**Type:** Practice / training exercise (not an official contest submission)
**Target:** SSSwap, a minimal Solana/Anchor automated market maker (AMM)
**Date of exercise:** August 2026

---

> ### Disclaimer
> This is a **practice/training exercise**, not an official audit engagement or a
> contest submission. The target code is from **CodeHawks "First Flight" #41 (SSSwap)**,
> a public, beginner-oriented educational security contest that **concluded in mid‑2025
> (May–June 2025)**. We did **not** participate in the live contest, made **no official
> submission**, and claim **no placement, ranking, or reward**. The official finding set
> was published after the contest closed; we use it here purely as a **known-answer key**
> to measure our internal tooling against an independent baseline. Nothing here should be
> read as an endorsement by, or affiliation with, the contest organizer or the original
> project author.

---

## Why this exercise

AMM/DEX math bugs (liquidity-share accounting, invariant preservation, rounding, decimal
handling) are a distinct failure class from the vault/lending/crowdfunding patterns most
checklists emphasize. We ran SSSwap through our audit pipeline as a controlled, known-answer
benchmark to (a) validate coverage on AMM-specific logic and (b) exercise a multi-layer
design where independent passes catch what a single pass misses.

## Methodology — a three-layer pipeline

Unlike a single-pass LLM review, findings flow through three independent stages:

1. **Primary analysis (AuditAgent).** A structured, checklist-driven large-language-model
   pass over the full in-scope source. For this exercise the checklist was extended with a
   dedicated AMM/DEX module (LP-token proportionality, constant-product invariant, deposit-
   vs swap-side slippage as *separate* checks, decimal handling, minimum-liquidity lock,
   reserve reload after transfers, integer-truncation on narrowing casts).

2. **Validation (Validator).** Reported issues are handed to an execution-based validator
   that attempts to construct a proof-of-concept and mark each claim as proven / unproven,
   rather than trusting the model's assertion. This is where non-applicable patterns get
   filtered instead of shipped.

3. **Second-opinion QA (independent reasoning pass).** A separate reasoning model re-examines
   the program with a state-machine / internal-consistency lens, explicitly tasked with
   finding *high-value bugs the first pass missed* — not restating existing findings.

The value of the design is redundancy across *different* reasoning styles: the second-opinion
layer independently rediscovered a core AMM accounting bug that the primary pass had framed
differently.

## Results

Measured against the published set of **5 official High-severity findings**, the pipeline
**identified 4 of the 5 Highs**, and surfaced several additional issues (a narrowing-cast
truncation and an initialization griefing vector) below the official High tier.

| # | Issue | Severity | Found by |
|---|-------|----------|----------|
| 1 | LP tokens minted for later deposits reuse the initial-deposit formula instead of a proportional (share-of-supply) calculation → dilution of existing providers | High | Second-opinion QA |
| 2 | `provide_liquidity` has no deposit-side slippage bound (independent of swap-side slippage) → add-liquidity can be sandwiched | High | Primary analysis |
| 3 | Reserve/vault balances are read without a reload after token transfers → share math can use stale state | High | Primary analysis |
| 4 | No minimum-liquidity lock on first deposit → first-depositor share-inflation / drainage vector | High | Primary analysis |
| 5 | Narrowing `u128 → u64` casts on intermediate amounts can silently truncate/wrap, and round-to-zero on liquidity removal | Medium | Primary + QA |
| 6 | Pool initialization can be griefed by pre-creating the pool's token accounts | Medium | Second-opinion QA |

**Coverage: 4 / 5 official High-severity findings, plus additional lower-severity findings
including via the second-opinion QA layer.** We deliberately do **not** claim "found every bug"
or "100% coverage."

## Limitations & ongoing development

- **One official High was not caught:** missing **decimal validation at pool initialization**
  (H‑03) — the pipeline did not flag that a pool can be created from two tokens with
  mismatched decimals. This is recorded as an open area of continued development; we are
  choosing **not** to over-tune the checklist against this single case, and will revisit it
  after evaluating a broader set of AMM samples.
- **The multi-layer design also catches its own noise:** the primary pass proposed one
  reentrancy/ordering-style issue that does not apply to Solana's execution model; the
  validation layer correctly declined to confirm it. We call this out to be transparent about
  false-positive handling rather than presenting a cherry-picked result.

## Takeaway

On a known-answer AMM benchmark, a three-layer pipeline (checklist-driven primary analysis →
execution-based validation → independent second-opinion reasoning) recovered 4 of 5
High-severity findings plus additional issues — with the second-opinion layer demonstrably
adding coverage a single pass missed. Remaining gaps (decimal validation at initialization)
are tracked and honestly disclosed.
