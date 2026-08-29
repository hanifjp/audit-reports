# Anchor Framework Security Corpus — Detection Benchmark (coral-xyz/sealevel-attacks)

**Author:** Hafiz Labs — internal capability showcase
**Type:** Practice / training exercise (not an official contest submission)
**Target:** `coral-xyz/sealevel-attacks` — the Anchor team's public corpus of common Solana/Anchor security exploits
**Date of exercise:** August 2026

---

> ### Disclaimer
> This is a **practice/training exercise**, not an official audit engagement or a contest
> submission. The target is **`coral-xyz/sealevel-attacks`**, a public, educational repository
> maintained by the Anchor framework team that documents common Solana/Anchor vulnerability
> classes. Each example ships an `insecure`, a `secure`, and a `recommended` variant, and the
> folder name states the intended issue — so the corpus effectively provides its own
> **known-answer key**. There is no contest, ranking, placement, or reward associated with it;
> we use it purely as an independent baseline to measure our own review process. Nothing here
> should be read as an endorsement by, or affiliation with, the Anchor framework team.

---

## Why this exercise

The eleven categories in this corpus are the canonical "you must handle these" failure modes of
the Solana programming model — missing signer/owner checks, account-data and type confusion,
unsafe initialization and closing, arbitrary CPI, PDA bump canonicalization and sharing, and
sysvar address validation. Because the intended issue for each example is documented, it is an
ideal **known-answer detection benchmark**: for every program there is an unambiguous "did we
catch the thing this example exists to teach?" question.

We ran the corpus through our review process to (a) measure detection coverage across these
canonical classes and (b) exercise our closed-loop improvement mechanism — where a gap the
process surfaces about itself is turned into a durable lesson and re-verified.

## Scope of the corpus

- **11 vulnerability categories**, each with an `insecure` variant (the program under review).
- **13 vulnerable programs in total** — one category (account closing) ships three progressively
  "still-vulnerable" variants to show that naïve fixes remain exploitable.
- The examples are deliberately **minimal, isolated snippets** (roughly 15–45 lines each): a
  single instruction demonstrating one issue, with no surrounding protocol, state, or realistic
  entrypoint.

## Methodology — a three-layer pipeline

Findings flow through three independent stages, the same design used in our other practice runs:

1. **Primary analysis.** A structured, checklist-driven large-language-model pass over the full
   in-scope source, producing typed findings with severity and location.

2. **Execution-based validation.** Reported issues are handed to a validator that attempts to
   build a runnable proof-of-concept and mark each claim proven / unproven, rather than trusting
   the model's assertion.

   > **Important note for this corpus:** these examples are minimal, non-runnable snippets with
   > no realistic entrypoint or state to exploit, so a compiled proof-of-concept is not
   > constructible for them. As expected, the validation layer returned **"unable to prove"** for
   > every example. That is the correct outcome for isolated teaching snippets and **not** a
   > detection failure — on this particular corpus the meaningful metric is **detection quality**,
   > not PoC-proving. (PoC-proving is exercised in our other, runnable practice targets.)

3. **Second-opinion QA.** A separate reasoning pass re-examines each program specifically to find
   high-value issues the first pass missed, rather than restating existing findings.

Throughout, the process monitors each stage for truncated or incomplete model output. Across all
13 programs, **every analysis completed in full with no token-exhaustion or silent-failure
cases** — so every "detected" and every "not-detected" below is a genuine, conclusive result, not
an artifact of a cut-off run. We call this out because distinguishing "analysed and missed" from
"never actually finished" is essential to an honest benchmark.

## Results — an honest two-stage account

### Stage 1 — initial pass: 10 of 11 categories detected

On the first run, the process correctly identified the intended vulnerability in **10 of the 11
categories**, each at an appropriate Critical/High severity:

| # | Category | Intended issue | Detected? | Severity |
|---|----------|----------------|-----------|----------|
| 1 | Signer authorization | Missing signer check | ✅ | Critical |
| 2 | Account data matching | Stored owner/authority field not validated | ✅ | High |
| 3 | Owner checks | Account owner not verified | ✅ | High |
| 4 | Type cosplay | Account type / discriminator confusion | ✅ | Critical |
| 5 | Initialization | Unchecked / arbitrary initialization | ✅ | Critical |
| 6 | Arbitrary CPI | Unverified program invoked via CPI | ✅ | Critical |
| 7 | Duplicate mutable accounts | Same account passed twice, unchecked | ✅ | Critical |
| 8 | Bump seed canonicalization | Non-canonical PDA bump accepted | ✅ | Critical |
| 9 | PDA sharing | Over-shared PDA / missing binding | ✅ | Critical |
| 10 | Closing accounts | Account revival after close | ⚠️ *initially missed — see below* | — |
| 11 | Sysvar address checking | Sysvar account address not validated | ✅ | Critical |

### The one gap — account closing

The account-closing category (all three "still-vulnerable" variants) is the one the initial pass
did **not** correctly resolve. It is worth being precise about *how* it missed:

- The process **did** report real, valid, adjacent problems on those programs — missing
  authorization on the close instruction, account aliasing / lamport-conservation issues when the
  same account is supplied twice, and missing owner validation.
- But it **missed the canonical vulnerability the category exists to teach**: *account revival
  after close*. Zeroing an account's lamports alone is insufficient; unless the account's data is
  also cleared and a closed-account discriminator is written, an attacker can refund the rent
  within the same transaction and keep the account "alive" with its old data intact. One finding
  even proposed a fix that does not address the revival vector.

So the honest first-pass result is **10 / 11 categories**, with account-closing identified as a
genuine coverage gap — not a false "we caught everything."

### Stage 2 — closed-loop improvement: gap → lesson → re-verified

Rather than leave the gap as a footnote, we treated it as the intended output of the exercise:

1. **Gap analysed.** We characterised exactly what was missed (the revival / missing
   data-zeroing + closed-discriminator semantics) and how it differs from the adjacent bugs that
   *were* found.
2. **Encoded as a durable lesson.** The pattern was added to our lesson-memory system — a store
   of curated, reusable security lessons that are retrieved and supplied to the review process as
   context when relevant code is analysed.
3. **Re-verified on the same example.** On re-running the closing-accounts program, the lesson
   was retrieved and injected into the analysis, and the process **now reports the account-revival
   vulnerability correctly** — describing it as failing to zero account data or set a closed
   discriminator, and citing the correct mitigation.

The sequence matters, and we state it plainly:

> **10 / 11 on the initial pass → the account-closing gap was surfaced → a lesson was added to
> the system → the same example is now detected correctly (11 / 11 after iteration).**

This is **not** a claim of "11 / 11 from the first run." It is a demonstration of **closed-loop
improvement**: the system surfaced a gap about its own coverage, learned from it, and verified the
fix on the exact case it had missed.

## Limitations & honest disclosure

- **The 11/11 is post-iteration, not initial.** The initial, un-assisted result was 10/11; the
  full-coverage result required adding a lesson and re-testing. Both numbers are reported above on
  purpose.
- **A single re-test is a demonstration, not a guarantee.** Detecting the revival case once after
  adding the lesson shows the mechanism works on that case; it is not a statistical claim of
  robustness across all future closing-account code.
- **These are minimal teaching snippets.** Detecting a documented issue in a 20-line isolated
  example is easier than surfacing the same class inside a large, realistic protocol. The corpus
  measures recognition of canonical patterns, not end-to-end audit difficulty.
- **No proof-of-concepts were produced here, by design** — the examples are non-runnable (see the
  methodology note). Detection, not PoC-proving, is the metric for this corpus.

## Takeaway

Against the Anchor team's own known-answer corpus of canonical Solana/Anchor vulnerability
classes, our review process detected the intended issue in **10 of 11 categories on the first
pass**, with full, non-truncated analysis on every one of the 13 programs. The single gap —
account revival on close — was surfaced honestly, encoded as a reusable lesson, and then detected
correctly on re-test, taking the corpus to **11 / 11 after one iteration**. We present this as
evidence of two things: solid baseline coverage of canonical Anchor failure modes, and a working
closed-loop process that turns a self-identified gap into a durable, verified improvement.
