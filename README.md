# 🛡️ Hafiz Labs — Smart Contract Audit Reports

> **Multi-Layer Verification for Solana Smart Contracts**
> Rigorous, Multi-Agent Security Analysis

[![Chain](https://img.shields.io/badge/Chain-Solana-blue)](https://github.com/hanifjp/audit-reports)
[![Methodology](https://img.shields.io/badge/Methodology-Multi--Layer-blue)](https://github.com/hanifjp/audit-reports)

---

## 🏢 About Hafiz Labs

**Hafiz Labs** is a Web3 security firm specializing in smart contract audits for the **Solana** ecosystem. Every audit runs through a multi-agent, multi-layer pipeline — automated analysis, execution-based validation, and an independent second-opinion review — with human expert verification of every finding before delivery.

### Why Hafiz Labs?

| Feature | Details |
|---------|---------|
| 🔬 **Multi-Layer Verification** | Analysis → execution-based validation → independent second-opinion review |
| 👁️ **Human-Verified** | Every finding reviewed by security experts |
| ⚡ **Fast Delivery** | Spot audit in 24-48 hours, Full audit in 3-5 days |
| 💰 **Competitive Pricing** | Enterprise quality at freelancer price |
| 🔍 **Comprehensive** | Business logic, access control, math invariants, Anchor best practices |

---

## 💼 Services & Pricing

We offer two paths depending on whether your code is public or private.

### Path A — Public repositories (standard tiers)

For code you can share as a **public GitHub repo** (or an equivalent zip of publicly-released
code), the standard tiers below apply.

| Service | Price | Delivery | Scope | Best For |
|---------|-------|----------|-------|---------|
| 🔍 **Spot Audit** | $150 USDC | 24-48 hours | Up to 150 lines **AND** 1-2 files | Quick security check before launch |
| 📋 **Full Audit** | $500 USDC | 3-5 days | Up to 500 lines **AND** 5 files | Comprehensive audit with full report |
| 🚨 **Emergency Audit** | $300 USDC | 6-12 hours | Up to 300 lines **AND** 2-3 files | Urgent audit for imminent launch |
| 💬 **Deep-Dive Consultation Report** | $75 USDC | 24-hour turnaround | Async / written — email-based Q&A, no live call required | Quick technical Q&A without full audit |

> 📐 **What counts toward scope — strict definition.**
> "File" and "lines" refer **only** to your Solana program's source logic: `.rs` files that
> contain your program logic (e.g. `lib.rs`, `state.rs`, `instructions/*.rs`).
> The following are **never counted** toward the limits:
> - **Test files** (e.g. `tests/`, `*_test.rs`, anything under a test module),
> - **Configuration files** (e.g. `Anchor.toml`, `Cargo.toml`, `Cargo.lock`),
> - **Auto-generated files** (e.g. the IDL, anything under `target/`).
>
> **Both limits must be satisfied — line count AND file count (AND, not OR). This applies to
> every tier.** For example, Spot must be within **both** 150 lines **and** 1–2 files;
> Emergency within **both** 300 lines **and** 2–3 files; Full within **both** 500 lines **and**
> 5 files. **If *either* the line count *or* the file count is exceeded, the job moves up to the
> next tier — or needs a custom quote if it exceeds the top tier.**

> 🚫 **Custom-quote exceptions (all tiers, automatic).**
> Regardless of line or file count, the following **always** require a custom quote that is
> **agreed in writing before work begins** — there is no mid-audit renegotiation, and these
> never fall under the fixed tiers above:
> - **External oracle integrations** (e.g. Pyth, Switchboard),
> - **Complex cross-program invocation (CPI)** to third-party programs,
> - **Custom or novel cryptography.**
>
> These involve substantial additional verification work, so they are scoped and priced up front.

> 🔁 **Spot & Full Audit include one free re-check** — after you apply the recommended fixes, we
> verify them (fix verification, not a full re-audit of new code). Emergency Audit does **not**
> include a free re-check.
>
> 💳 **Payment terms:** Full payment upfront via USDC before work begins.
>
> ⏱️ **Delivery timeframes are targets, not guarantees.** The turnaround times above (24-48h,
> 3-5 days, 6-12h) and the free re-check depend on the final agreed scope. They do **not** apply
> to custom-quote exceptions (external oracle integrations, complex CPI, or novel cryptography),
> which are scheduled separately as part of the custom quote.
>
> 💡 Focused exclusively on Solana/Anchor — deep specialization over broad, generic coverage.

### Path B — Private code

> ℹ️ **Status: by request only.** A dedicated private-code audit track is **not yet generally
> available**. We can take private-code engagements **on request, only after a written agreement
> is signed**, and each is scoped and priced individually (**custom quote**).
>
> We deliberately make **no promises we cannot yet prove.** In particular, for private-code
> engagements we do **not** currently claim any specific data-retention period, encryption
> standard, or other handling guarantee beyond what is written in the signed agreement for that
> engagement. Any such terms, if offered, will be stated explicitly in that agreement — not
> implied here.
>
> To discuss a private-code engagement, contact us (see **How to Order**) and we will send scope
> and pricing for your specific case.

---

## 🧪 Sample Reports

These are **example reports generated on synthetic test contracts** — not real client
audits — to demonstrate the format and depth of a Hafiz Labs professional-tier report.
The contracts are intentionally vulnerable teaching examples; no client code, names, or
data are involved.

| Sample | Domain | Findings | Notes | Format |
|--------|--------|----------|-------|--------|
| [DeFi Crowdfunding](./sample-reports/sample-report-defi-crowdfunding.pdf) | Fund integrity | 2 Critical · 1 High · 1 Medium | Synthetic test contract — not a real project | PDF |
| [NFT Marketplace](./sample-reports/sample-report-nft-marketplace.pdf) | Access control | 3 High | Synthetic test contract — not a real project | PDF |

Each sample demonstrates our professional report format:

- 📊 **CVSS 3.1 scoring** — every finding scored with a full base vector and severity band
- 🩹 **Vulnerable → Fixed code diff** — the exact insecure snippet and a concrete remediation
- 🎯 **Exploit walkthrough** — a step-by-step attack scenario for each issue
- 🔬 **Honest PoC status** — the raw automated-validation result is shown verbatim; when an
  exploit could not be auto-proven, we say so plainly rather than implying a proof we don't have
- ⚖️ **Recommended Fix Order** — findings prioritized by CVSS with a suggested remediation timeframe

> These samples reflect the AI-assisted, human-verified pipeline described above; they are
> provided for demonstration and educational purposes only.

---

## 🎯 Practice Runs (Public Benchmarks)

**Practice / training exercises** — we run our pipeline against **public** security-contest
codebases and labelled vulnerability corpora. These are **not** paid engagements, official
contest submissions, or client audits, and we claim **no placement, ranking, or reward**. They
exist to benchmark and improve our tooling in the open.

> ⚠️ **How to read these numbers.** Results are **self-assessed** by us against each target's
> published official findings or our own earlier baseline (as noted per row) — they are **not**
> independently verified by a third party. Some of these repositories were also used to **tune
> our system** (e.g. adding lessons to our lesson-memory store), so later results on the same
> repository are not blind.

| Exercise | Target | Result |
|----------|--------|--------|
| [SSSwap — Solana AMM](./practice-runs/ssswap-solana-amm.md) | Minimal Solana AMM · CodeHawks "First Flight" educational contest (closed mid‑2025) | Identified **4 of 5** official High-severity findings + additional lower-severity findings via the multi-layer pipeline |
| [Anchor Framework Security Corpus](./practice-runs/sealevel-attacks-benchmark.md) | `coral-xyz/sealevel-attacks` · Anchor team's public corpus of 11 canonical Solana/Anchor vulnerability classes (known-answer key) | **10 of 11** categories detected on the first (blind) pass. *Process note (not a headline score): after the one gap — account revival on close — was turned into a lesson and re-tested on the same example, that case is now detected too; this post-iteration result is not blind.* |
| [RustFund Re-Audit](./practice-runs/rustfund-relearning.md) | CodeHawks "Rust Fund" First Flight (closed 2025) | Measured against **our own earlier internal baseline** (self-comparison, not a published answer key): reproduced **5 of 6** of those baseline findings and surfaced **1 new high-value finding** the earlier baseline missed |

> Each practice run is written up with honest framing — **including findings our pipeline did
> _not_ catch** — as a transparent record of ongoing development, not a marketing claim.

---

## 🔍 Audit Methodology

Our audit process covers **4 critical domains**:

### 1. 🏗️ Business Logic Analysis
Identify flaws in transaction flows, asset handling, and protocol logic that could lead to economic exploits or fund loss.

### 2. 🔐 Access Control Audit
Verify that every state-changing function has proper `Signer` constraints, authority checks, and ownership validation.

### 3. 📐 Mathematical Invariant Check
Detect arithmetic overflow/underflow risks, precision loss in calculations, and invariant violations that could be exploited.

### 4. ⚓ Anchor Account Best Practices
Validate correct usage of `seeds`, `bump`, `authority`, `close`, and `has_one` constraints in Anchor programs.

---

## 📊 Report Format

Every Hafiz Labs audit report includes:

- ✅ **Executive Summary** — Professional security assessment
- ✅ **Vulnerability Table** — ID, Severity, Location, Description
- ✅ **Remediation Plan** — Specific fix code for each finding
- ✅ **Attack Scenario** — Step-by-step exploit walkthrough
- ✅ **Additional Metrics** — Time-to-fix, exploit probability, lines of code

---

## 🚀 How to Order

1. Contact us via email: contact.hafizlabs@gmail.com (or open a GitHub Issue)
2. **Share your code** via one of the following:
   - A public GitHub repo link (Path A — standard tiers), or
   - A `.zip` file of your **publicly-released** project attached to the email (Path A), or
   - For **private code** (Path B): contact us first — a private engagement proceeds only after
     a written agreement is signed, and is scoped individually.
3. **Choose your package** (Spot / Full / Emergency) — or request a custom quote.
4. **Pay via USDC** on Solana — full payment upfront before work begins
5. **Receive your report** within the promised timeframe

> ⏱️ **We typically respond within 24 hours.**

---

## 🔒 Privacy & Data Handling

We state only what we actually do today.

- **Please do not paste private source code into chat.** For any code you want kept private, use
  the Path B process above (contact us; proceed under a signed agreement). Do not send private
  code through chat channels.
- **Public code is processed with AI providers.** Our pipeline uses third-party AI providers to
  analyze code. For that processing we **request zero-data-retention settings** from the
  provider. We cannot independently guarantee the provider's internal behavior, so we describe
  this as the setting we request — **not** as a warranty.
- **No unverified claims.** Beyond the above, we do not claim any specific encryption standard,
  retention period, or other data-handling guarantee unless it is written into a signed
  agreement for your specific engagement.

---

## 🛡️ Severity Classification

| Level | Description | Action Required |
|-------|-------------|-----------------|
| 🔴 **CRITICAL** | Funds at immediate risk, exploitable now | Fix before mainnet |
| 🟠 **HIGH** | Significant vulnerability, likely exploitable | Fix before mainnet |
| 🟡 **MEDIUM** | Moderate risk, may be exploitable under conditions | Fix recommended |
| 🟢 **LOW** | Minor issue, best practice violation | Fix when possible |

---

## 📄 Engagement Terms

Engagement terms are agreed in writing before work begins.

---

## 📞 Contact

| Channel | Details |
|---------|---------|
| 🌐 **GitHub** | [github.com/hanifjp/audit-reports](https://github.com/hanifjp/audit-reports) |
| 💬 **Telegram** | Coming soon |
| 📧 **Email** | contact.hafizlabs@gmail.com |

---

## ⚠️ Disclaimer

Hafiz Labs audit reports are not a guarantee of security. Smart contracts may contain vulnerabilities that were not identified during the audit. Always follow security best practices and consider multiple audits for high-value protocols.

---

*© 2026 Hafiz Labs. All rights reserved.*

> 🔒 The sample and practice-run reports in this repository are public and provided for educational purposes. Client deliverables are shared only with the client.
