# PRD: Contract Review Agent (NDA + MSA)

| Field | Value |
|-------|-------|
| **Product** | Contract Review Agent for SMB Legal |
| **Author** | Senior Product Manager |
| **Audience** | Engineering, Legal/Compliance, Design |
| **Status** | Draft — decision-ready for MVP go/no-go |
| **Version** | 0.1 |
| **Last updated** | 2026-06-18 |

---

## 1) Executive Summary

The Contract Review Agent is a B2B SaaS product that reviews **inbound vendor/customer NDAs and MSAs** against a company's **approved playbook**, flags risky or non-standard clauses, explains why each is a problem, and proposes **suggested redlines** that a human approves before use.

The target buyer is an in-house counsel or operations lead at a company **too small for a large legal team but unable to skip contract review**. Today these teams read contracts line by line, apply inconsistent standards across deals, and risk non-lawyers signing dangerous terms.

Our approach uses **RAG grounded on the customer's own playbook** (so the agent cites the customer's standard rather than inventing legal advice) inside a **planner-executor pipeline** (segment contract → identify clause types → check each against the playbook → propose redlines). A **human-in-the-loop** approval step is mandatory; the product never auto-finalizes language.

**Why now (2026):** LLM evaluation/governance has matured enough to measure and prove clause-level accuracy, and retrieval/workflow standardization makes a company's playbook a reusable, structured asset.

**MVP (6–8 weeks):** NDA + MSA only, a playbook builder that infers standards from past contracts, a clause-flagging + redline review UI, validated with **3 design partners**. Commercial model = **per-contract usage fee + monthly SaaS base fee**.

**Success:** (1) cut review time per contract by **50%**; (2) increase the **% of contracts where playbook violations are caught** before signature.

---

## 2) Problem Statement

Small and mid-size companies sign NDAs and MSAs constantly (sales deals, vendor onboarding, partnerships), but they lack the legal capacity to review each one properly.

**Concrete pain:**
- **Manual line-by-line review is slow.** A single reviewer (often the only in-house lawyer or an ops lead) is the bottleneck; contracts wait in a queue, slowing deals.
- **Non-lawyers sign risky terms.** When legal is too slow, sales/ops sign NDAs/MSAs themselves and unknowingly accept uncapped liability, perpetual IP licenses, or auto-renewals.
- **Inconsistent standards across deals.** Different reviewers (or the same reviewer on different days) apply different thresholds, so the company's risk posture is incoherent and hard to audit.

**Cost of the problem:** delayed revenue (deal velocity), legal exposure (bad terms signed), and wasted senior-person time on repetitive low-value first-pass reading.

---

## 3) Goals and Non-Goals

### Goals
- **G1.** Reduce median review time per NDA/MSA by **≥50%** vs. the team's current manual baseline.
- **G2.** Increase the share of contracts where a **playbook violation is caught and surfaced** before signature.
- **G3.** Produce **grounded, cited** flags and redlines that always reference a specific playbook rule (no ungrounded "legal advice").
- **G4.** Keep a human **explicitly in control**: every redline is reviewed/accepted/edited/rejected by a person, with a full audit trail.
- **G5.** Ship an MVP validated with **3 design partners** in 6–8 weeks.

### Non-Goals (explicit)
- **NG1.** **No contract types beyond NDA and MSA** in MVP (no DPAs, SOWs, employment, leases, financing docs).
- **NG2.** **No autonomous signing or sending.** The product never executes, e-signs, or transmits a contract to a counterparty.
- **NG3.** **Not a replacement for a lawyer.** It is a first-pass assistant; it does not provide legal advice or guarantee legal sufficiency.
- **NG4.** **No multi-language / non-English contracts** in MVP.
- **NG5.** **No negotiation automation** (no agent-to-counterparty back-and-forth) in MVP.
- **NG6.** **No CLM (contract lifecycle management) suite** — no obligation tracking, renewals calendar, or e-signature integration in MVP.
- **NG7.** **No jurisdiction-specific legal interpretation engine** beyond the assumptions encoded in the customer's playbook.

---

## 4) Target Users & Personas

| Persona | Role | Goals | Frustrations | Relationship to product |
|---------|------|-------|--------------|--------------------------|
| **Dana — Solo In-House Counsel** | Only lawyer at a 150–800 person company | Review more contracts without hiring; enforce consistent standards | Drowning in NDAs/MSAs; context-switching; repetitive first-pass reading | **Primary user + economic buyer.** Builds/owns the playbook; approves redlines. |
| **Marco — Ops/Deal Lead** | RevOps, procurement, or BizOps | Close deals fast; not get blocked by legal | Waits days for legal; sometimes signs without review | **Secondary user.** Uploads contracts, triages low-risk ones, escalates flagged ones. |
| **Priya — Compliance/Security Lead** | GRC owner | Auditable, consistent process; data control | No record of why a term was accepted; AI tools that leak data | **Approver/gatekeeper.** Cares about audit log, RBAC, retention. |
| **Sam — Sales Rep** (peripheral) | AE | Get the customer's NDA/MSA cleared quickly | Legal is a black box | Submits contracts; sees status, not internals. |

**Assumption (explicit):** The buyer is Dana (counsel) or her budget; Marco is the volume driver of usage.

---

## 5) User Stories & Key User Flows

### User Stories (≥8)
1. **As Dana**, I want to upload an inbound NDA and get flagged clauses with explanations, so I can review only what deviates from our playbook.
2. **As Dana**, I want each flag to cite the exact playbook rule it violates, so I can trust and verify the AI's reasoning.
3. **As Dana**, I want suggested redline text I can accept, edit, or reject, so I keep control of final language.
4. **As Dana**, I want to build our playbook by uploading past "good" contracts, so I don't have to write every standard from scratch.
5. **As Dana**, I want to review and approve the inferred playbook standards, so nothing becomes a "rule" without my sign-off.
6. **As Marco**, I want to upload a vendor MSA and see an overall risk summary, so I know whether I can self-serve or must escalate to Dana.
7. **As Marco**, I want clear status (in review / flagged / cleared), so I can tell sales when a contract is good to go.
8. **As Priya**, I want a full audit log of who accepted which redline and when, so we can defend our decisions later.
9. **As Priya**, I want role-based access so reps can submit but only counsel can approve redlines.
10. **As Dana**, I want to export a redlined document (and a summary memo) in Word/PDF, so I can send it to the counterparty outside the tool.
11. **As Dana**, I want versioning so I can see how a contract changed across review rounds.
12. **As Priya**, I want assurance that our contracts and playbook are not used to train external models and are deleted per our retention policy.

### Key User Flows

**Flow A — First-time playbook setup (Dana)**
1. Create workspace → choose contract type scope (NDA, MSA).
2. Upload 5–20 past accepted contracts.
3. System infers candidate standards per clause category (with proposed thresholds + "approved language").
4. Dana reviews each candidate standard → accept / edit / reject.
5. Playbook v1 published and versioned.

**Flow B — Contract review (Marco → Dana)**
1. Marco uploads an inbound NDA/MSA (PDF/DOCX).
2. System parses → segments clauses → classifies clause types → retrieves matching playbook rules → scores deviation/risk → drafts redlines.
3. Review UI shows: risk summary, flagged clauses (with citation + explanation), suggested redlines.
4. Low-risk → Marco clears; flagged/high-risk → routed to Dana.
5. Dana accepts/edits/rejects each redline (human-in-the-loop).
6. Export redlined DOCX + summary memo; version saved; audit log updated.

**Flow C — Audit/review (Priya)**
1. Open a contract's history → see every flag, decision, editor, timestamp, and playbook version used.
2. Export audit report.

---

## 6) MVP Scope (In / Out)

### In Scope
- **Contract types:** NDA and MSA **only** (English, machine-readable PDF/DOCX; OCR fallback for scans).
- **Playbook builder:** infer standards from uploaded past contracts + manual review/accept.
- **Review pipeline:** upload → parse → segment → classify → retrieve/ground → deviation/risk score → redline suggestion.
- **Review UI:** risk summary, per-clause flags with citations + explanations, accept/edit/reject redlines.
- **Human approval workflow:** role-based; only authorized approvers finalize redlines.
- **Export & versioning:** redlined DOCX + summary memo; contract version history; audit log.
- **Commercial:** per-contract usage metering + monthly SaaS base fee; usage dashboard.
- **Design-partner program:** 3 companies, real (anonymizable) contracts.

### Out of Scope (MVP)
- Other contract types (DPA, SOW, employment, leases) — see NG1.
- Autonomous send/e-sign/negotiation — NG2, NG5.
- Non-English / multi-jurisdiction interpretation — NG4, NG7.
- Native CRM/CLM/e-signature integrations (manual export only) — NG6.
- Mobile app (responsive web is sufficient).
- Bulk/batch processing pipelines beyond single-contract upload.

---

## 7) Functional Requirements

> Scope note: All requirements below apply to **NDA and MSA contract types only** for MVP.

| Feature | User Value | Detailed Requirement | Dependencies | Acceptance Criteria |
|---------|-----------|----------------------|--------------|---------------------|
| **Contract upload & parsing** | Get a contract into the system reliably | Accept PDF/DOCX (≤25MB). Extract text with layout awareness; OCR fallback for scanned PDFs; detect language and reject non-English with a clear message; flag low-confidence OCR pages. | OCR engine; document parser; file storage | Given a machine-readable NDA/MSA, text extraction ≥99% character accuracy on a labeled test set; scanned doc triggers OCR and surfaces a confidence warning; non-English upload is rejected gracefully. |
| **Clause segmentation** | Review per clause, not whole-doc | Split contract into discrete clauses/sections with stable IDs and source character offsets for citation/highlighting. | Parsing output | ≥90% clause boundary F1 vs. annotated NDA/MSA gold set; each clause maps back to exact source location. |
| **Clause classification (NDA+MSA)** | Know which playbook rule applies | Classify each clause into a fixed taxonomy (e.g., confidentiality scope, term/termination, liability cap, indemnification, IP ownership/license, auto-renewal, governing law, assignment, warranties). | Segmentation; clause taxonomy | ≥85% top-1 classification accuracy on gold set; unknown/low-confidence clauses labeled "unclassified" rather than mis-bucketed. |
| **Playbook ingestion** | Reuse company standards | Ingest accepted standards per clause category: approved language, thresholds, hard/soft constraints, jurisdiction assumptions. Store versioned. | Playbook Builder (§9); vector store | A published playbook is queryable by clause category; each rule has an immutable version ID. |
| **Retrieval / grounding (RAG)** | Trustworthy, cited output | For each classified clause, retrieve the relevant playbook rule(s) and ground all flags/redlines in retrieved text; every flag must carry a citation to a specific playbook rule + contract span. | Playbook ingestion; embeddings; vector store | 100% of flags include a resolvable citation (playbook rule ID + contract char span); no flag is emitted without retrieved grounding. |
| **Deviation / risk scoring** | Prioritize attention | Compare clause to retrieved rule; output {compliant, deviation} + severity (low/med/high) + plain-language explanation of the risk. | RAG; scoring rubric | On gold set, deviation detection recall ≥0.85 for high-severity categories (liability, IP, auto-renewal); severity calibrated against design-partner counsel labels. |
| **Redline suggestion** | Save drafting time | For each deviation, propose replacement language grounded in the playbook's approved wording; show a clear diff vs. original; mark as **suggested only**. | Deviation scoring; playbook approved language | Each high/med deviation yields a redline diff; ≥70% of suggested redlines accepted-or-minor-edited by design-partner counsel (tracked). |
| **Review UI** | Efficient human review | Single screen: risk summary header, clause list with filters (severity, category, status), per-clause citation + explanation + redline diff, accept/edit/reject controls. | All above | Counsel can review a flagged clause and act (accept/edit/reject) in ≤3 clicks; original ↔ redline ↔ playbook citation are visible together. |
| **Human approval workflow** | Keep humans in control + RBAC | Roles: Submitter, Reviewer, Approver, Admin. Only Approver/Admin finalize redlines. Track each decision (actor, timestamp, action) per clause. Nothing is "final" without explicit approval. | RBAC; audit log | A Submitter cannot finalize a redline (enforced server-side); every decision is logged immutably; contract cannot be marked "reviewed" with unresolved high-severity flags unless explicitly overridden + logged. |
| **Export & versioning** | Use output externally; defensibility | Export redlined DOCX (tracked changes) + summary memo (PDF). Maintain version history per contract and link the playbook version used. | Document generator; storage | Exported DOCX opens in Word with tracked changes intact; each review round is a retrievable version; export records which playbook version was applied. |

---

## 8) Non-Functional Requirements

| Requirement | Why it matters | Target | Verification method |
|-------------|----------------|--------|---------------------|
| **Grounding / citation integrity** | Ungrounded "legal advice" is the core risk | 100% of flags and redlines carry a resolvable citation to a playbook rule + contract span; 0 ungrounded assertions | Automated guardrail check in pipeline + eval harness on gold set; block emit if citation missing |
| **Clause-level accuracy** | Wrong flags erode trust | High-severity deviation recall ≥0.85; classification top-1 ≥0.85; false-positive flag rate ≤15% | Offline eval against counsel-labeled gold set; tracked per release |
| **End-to-end latency** | Reviewer flow must feel responsive | Standard NDA (≤10 pages) full analysis ≤60s p50 / ≤120s p95; MSA (≤40 pages) ≤180s p95 | Load test + production APM percentile tracking |
| **UI responsiveness** | Review must be fluid | Review actions (accept/edit/reject) reflect ≤300ms p95 | Front-end performance monitoring |
| **Security & audit logging** | Legal data is sensitive; defensibility | Encryption in transit (TLS 1.2+) and at rest (AES-256); immutable, append-only audit log of all access + decisions | Security review; pen test; audit-log tamper test |
| **Permissions / RBAC** | Prevent non-lawyers finalizing terms | Server-enforced roles (Submitter/Reviewer/Approver/Admin); least privilege | Authz test suite; negative tests (Submitter cannot approve) |
| **Data retention & isolation** | Buyer (Priya) requires control + no training leakage | Per-tenant data isolation; configurable retention (default 12 months) + hard delete on request; contracts/playbooks **excluded from any model training** | Tenant isolation test; documented data-flow; contractual + technical no-train guarantee; deletion verification |
| **Robustness to OCR/format issues** | Real contracts are messy scans | Detect scanned/low-quality docs; surface per-page OCR confidence; degrade gracefully (warn, never silently mis-read) | Test corpus of scanned/odd-format NDAs/MSAs; verify warnings appear; no silent failure |
| **Availability** | Blocks deal flow if down | 99.5% monthly uptime (MVP) | Uptime monitoring/SLO dashboard |
| **PII / confidentiality handling** | Contracts contain sensitive parties | Redact/secure logs (no contract text in plaintext app logs); access on need-to-know | Log inspection; security review |

---

## 9) Playbook Builder Requirements

The playbook is the **source of truth** the agent grounds on. Quality here determines product quality.

### What the playbook contains
- **Clause categories** (NDA + MSA taxonomy): confidentiality scope & duration, permitted use, term/termination, **liability cap**, indemnification, **IP ownership/license**, **auto-renewal**, governing law/jurisdiction, assignment, warranties/disclaimers, data/security obligations.
- **Approved language:** the company's preferred/fallback wording per category (primary + acceptable alternatives).
- **Thresholds / constraints:** quantitative and categorical limits, e.g.:
  - Liability cap ≤ 12 months of fees (hard); uncapped liability = hard-block.
  - Confidentiality term: 3–5 years acceptable; perpetual = flag.
  - Auto-renewal: allowed only with ≥30-day opt-out notice.
  - IP: no assignment of customer-created IP; license must be non-exclusive + revocable.
  - Constraints typed as **hard (must)** vs **soft (prefer)**.
- **Jurisdiction assumptions:** default governing law/venue the standards assume (e.g., Delaware, US); flag contracts outside assumed jurisdictions as "review carefully — playbook may not apply."

### How inferred standards are validated/accepted
1. **Infer:** From uploaded past accepted contracts, extract per-category candidate standards (most common accepted wording + observed thresholds), each with **supporting evidence** (which source contracts/clauses it was derived from) and a **confidence score**.
2. **Human review (required):** Dana sees each candidate standard with evidence and must **accept / edit / reject**. **No inferred standard becomes active without explicit human acceptance.**
3. **Classify constraint type:** Dana marks each as hard vs soft and sets thresholds.
4. **Publish & version:** Accepted standards form Playbook vN (immutable version ID). Subsequent edits create new versions; reviews record which version was applied.
5. **Conflict handling:** If inferred standards conflict across source contracts, surface the conflict and require Dana to choose the canonical standard.

**Assumption (explicit):** Design partners can provide ≥5 past accepted NDAs and ≥5 MSAs each to seed inference; otherwise the playbook is built manually from templates.

---

## 10) Risk & Compliance Considerations

### Limitations of AI advice
- The product is a **first-pass assistant, not legal advice** and does not guarantee legal sufficiency or enforceability.
- Output quality is bounded by playbook quality and document parse quality.

### Legal disclaimers (in-product)
- Persistent disclaimer in the review UI and on every export: *"AI-generated suggestions grounded on your playbook. Not legal advice. Human review required before use."*
- Onboarding acknowledgment that counsel retains responsibility for final terms.

### Escalation triggers (auto-route to counsel / block self-serve)
- Any **hard-constraint violation** (e.g., uncapped liability, perpetual IP assignment).
- **Unclassified** clauses or **low classification/grounding confidence**.
- Contract **outside assumed jurisdiction**.
- **Low OCR confidence** pages.
- Clause type **present in contract but absent from playbook** (no rule to ground on).

### Audit trail
- Immutable, append-only log: upload, parse/OCR confidence, every flag, every human decision (actor + timestamp + action), playbook version used, exports. Exportable as an audit report (supports Priya's defensibility need).

### Preventing confident-but-wrong redlines
- **Grounding guardrail:** the pipeline **must not emit** a flag/redline without a resolvable playbook citation (no citation → no output).
- **Abstain over guess:** low-confidence clauses are labeled "needs human review," not auto-redlined.
- **Suggested-only framing:** redlines are always diffs requiring explicit human accept; never applied silently.
- **Multi-check on high-severity items:** a verification pass re-checks high-severity redlines against the cited rule before display.
- **Calibration loop:** track accept/edit/reject rates per clause category; categories with high reject rates are downgraded to "flag only, no auto-redline" until improved.
- **Eval gating:** releases blocked if grounding integrity <100% or high-severity recall regresses below threshold on the gold set.

---

## 11) Metrics & Measurement Plan

| KPI | Definition | Baseline needed | Target | How measured (product analytics) |
|-----|-----------|-----------------|--------|----------------------------------|
| **Review time reduction (G1)** | Median minutes from contract upload → "reviewed/approved" status | Per design partner: capture current manual review time (intake survey + timed control set) | **−50%** vs. baseline | Timestamp delta between upload and final-decision events; compare cohort vs. baseline control set |
| **Playbook violation catch rate (G2)** | % of reviewed contracts where ≥1 true playbook violation was surfaced and acted on | Estimate current catch rate via retrospective sample reviewed manually | **Increase vs. baseline** (set numeric target per partner after baseline) | Count contracts with ≥1 confirmed-true flag (counsel-validated) / total reviewed |
| **Grounding integrity** | % of flags/redlines with resolvable citation | N/A (must be 100% by design) | **100%** | Automated pipeline guardrail metric + sampling audit |
| **Redline acceptance rate** | % of suggested redlines accepted or minor-edited | None | **≥70%** | Track accept/edit/reject events per redline |
| **False-positive flag rate** | % of flags counsel marks "not a real issue" | None | **≤15%** | Reviewer "reject as false positive" events / total flags |
| **High-severity recall** | Of known high-severity violations (gold/audited set), % caught | Build labeled audit set with design partners | **≥0.85** | Offline eval harness per release + periodic prod audits |
| **Time-to-first-value** | Time from signup → first contract reviewed | None | ≤1 business day | Onboarding funnel events |
| **Self-serve clearance rate** | % low-risk contracts cleared by ops without escalation | None | Trend up (efficiency) | Ratio of ops-cleared vs. escalated contracts |
| **Commercial: usage & retention** | Contracts/month per account; logo retention | None | Growth MoM; 3/3 design partners convert | Usage metering + billing analytics |

**Measurement assumptions (explicit):** Each design partner completes a **baseline capture** (manual review time + retrospective violation sample) during week 1; without baselines, G1/G2 cannot be proven, so this is a launch prerequisite.

---

## 12) Rollout Plan

### Phase 0 — Foundations (Weeks 1–2)
- Lock NDA/MSA clause taxonomy with design-partner counsel.
- Stand up parsing/OCR, segmentation, vector store, audit log, RBAC skeleton.
- Each design partner signs data agreement + completes **baseline capture** (G1/G2 prerequisite).

### Phase 1 — Design-partner alpha (Weeks 3–5)
- Playbook Builder + review pipeline + review UI behind a flag.
- Seed playbooks from partners' past contracts; counsel validates standards.
- Run real contracts; collect accept/edit/reject + gold-set labels; tune scoring/grounding.
- **Gate to MVP:** grounding integrity = 100%; high-severity recall ≥0.85; ≥70% redline acceptance on a sample.

### Phase 2 — MVP launch (Weeks 6–8)
- Export/versioning, usage metering, billing (per-contract + monthly base), usage dashboard.
- Harden security/retention; complete security review.
- Convert 3 design partners to paid; publish disclaimers + onboarding.
- **Launch criteria:** all §8 NFR targets met; ≥1 partner hits −50% review time on their control set.

### Phase 3 — Iteration (post-MVP)
- Expand clause taxonomy depth; calibration loop on high-reject categories.
- Candidate next bets (not committed): batch upload, CRM/CLM/e-sign integrations, additional contract types (DPA/SOW), multi-jurisdiction handling.
- Establish weekly eval review; ratchet accuracy gates upward.

---

## 13) Open Questions

1. **Baselines:** Can all 3 design partners provide reliable current review-time data and a retrospective violation sample? If not, how do we credibly prove G1/G2?
2. **Playbook seed volume:** Do partners have ≥5 NDAs and ≥5 MSAs of clean past contracts each, or do we need a template-based cold-start?
3. **Jurisdiction scope:** Which governing-law assumptions do we encode for MVP (US/Delaware only)? How do we handle UK/EU contracts partners may receive?
4. **MSA complexity ceiling:** MSAs vary widely (some bundle SOW/DPA terms). What's the max length/complexity we support before routing to "manual review"?
5. **Model hosting & data residency:** Which LLM provider(s), and can we meet a no-training + data-residency guarantee partners may demand? Is a self-host/VPC option needed for any partner?
6. **OCR threshold:** What OCR confidence cutoff triggers a hard block vs. a soft warning?
7. **DOCX redline fidelity:** Is Word tracked-changes export sufficient, or do partners need redlines inside their existing tooling (e.g., Google Docs, iManage)?
8. **Liability/disclaimer posture:** What contractual liability cap and disclaimer language does our own legal/compliance require for shipping AI-suggested redlines?
9. **Pricing calibration:** What per-contract price + monthly base do design partners validate as fair vs. their current cost-per-review?
10. **"Override" governance:** When counsel overrides a high-severity block, what approval level and logging are required to satisfy compliance?
11. **Confidence display:** Do we expose numeric confidence scores to users, or only categorical severity (to avoid false precision)?
12. **Gold-set ownership:** Who maintains the labeled evaluation set over time, and how do we avoid overfitting to the first 3 partners' preferences?

---

## Appendix A — Generation Prompt

This PRD was generated from the following prompt.

> You are a senior Product Manager writing a Product Requirements Document (PRD) for a B2B AI product. Use the input idea below to produce a complete, decision-ready PRD.
>
> **INPUT IDEA**
> - Product concept: An agent that reviews inbound vendor/customer contracts against a company's approved playbook, flags risky clauses, and proposes suggested redlines.
> - Target customer: In-house counsel or ops leads at companies too small for a large legal team but cannot skip review.
> - Customer pain: manual line-by-line review; non-lawyers signing risky terms; inconsistent standards across deals.
> - Solution: Upload a contract; compare it to an approved playbook; flag deviations (e.g., liability caps, auto-renewal, IP terms) with explanations; suggest redline language aligned to the company's standards.
> - LLM approach: RAG grounded on the company's own playbook; planner-executor pipeline (segment contract → check each clause type); human-in-the-loop acceptance for redlines.
> - Why now (2026): improved LLM evaluation/governance maturity and retrieval/workflow standardization.
> - MVP (6–8 weeks): support NDAs + MSAs only; include a playbook builder (upload past contracts to infer standards); clause flagging + redline suggestions in a review view; 3 design-partner companies; commercial pricing = per-contract usage + monthly SaaS base fee.
> - Difficulty: intermediate (narrow doc types reduce risk; grounding is understood).
> - Success metrics: (1) reduce review time per contract by 50%; (2) increase % of contracts that catch playbook violations.
>
> **PRD OUTPUT REQUIREMENTS (follow exactly)**
> 1. Executive Summary
> 2. Problem Statement
> 3. Goals and Non-Goals (include explicit Non-Goals)
> 4. Target Users & Personas
> 5. User Stories (at least 8) and Key User Flows
> 6. MVP Scope (what's in / out)
> 7. Functional Requirements (table with: Feature, User Value, Detailed Requirement, Dependencies, Acceptance Criteria)
>    - Must cover: contract upload/parsing, clause segmentation, playbook ingestion, retrieval/grounding, deviation/risk scoring, redline suggestion, review UI, human approval workflow, export/versioning.
>    - Must explicitly cover NDA + MSA only.
> 8. Non-Functional Requirements (table with: Requirement, Why it matters, Target, Verification method)
>    - Include: accuracy/citation expectations, latency targets, security/audit logging, permissions/role-based access, data retention controls, robustness to OCR/format issues.
> 9. Playbook Builder Requirements
>    - Define what the playbook contains (clause categories, "approved language," thresholds, jurisdiction assumptions).
>    - Include how inferred standards are validated/accepted.
> 10. Risk & Compliance Considerations
>     - Include: limitations of AI advice, legal disclaimers, escalation/escalation triggers, audit trail, and how you prevent confident-but-wrong redlines.
> 11. Metrics & Measurement Plan
>     - For each KPI: definition, baseline needed, target, and how you'll measure it in product analytics.
> 12. Rollout Plan (design partners → MVP launch → iteration)
> 13. Open Questions
>
> **CRITICAL CONSTRAINTS**
> - Keep assumptions explicit. If you need info not provided, list it under Open Questions.
> - The PRD must be written for internal use by engineering, legal/compliance, and design.
> - Do not give generic PRD advice; every section should tie back to the contract-review agent.
>
> Write the PRD now.
