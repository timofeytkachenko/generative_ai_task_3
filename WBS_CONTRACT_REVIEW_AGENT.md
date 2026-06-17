# WBS: Contract Review Agent MVP (6–8 Weeks)

| Field | Value |
|-------|-------|
| **Product** | Contract Review Agent for SMB Legal |
| **Scope** | NDA + MSA only; playbook builder; clause flagging + redline suggestions; 3 design partners |
| **Architecture** | RAG grounding on playbook; planner-executor (segment → check each clause); human-in-the-loop redlines |
| **Owner roles** | Product (PM), ML Engineer (ML), Backend Engineer (BE), Frontend Engineer (FE), UX, Legal Ops (Legal), QA, DevOps |
| **Target** | Ship MVP in 6–8 weeks; −50% review time; ↑ violation catch rate |

---

## Work Breakdown Structure

### 1. Program Setup & Design-Partner Onboarding

#### 1.1 Design-partner program
- **1.1.1 Recruit & sign 3 design partners**
  - **Deliverable:** 3 signed design-partner agreements + data processing agreements (DPAs)
  - **Owner:** Product
  - **Dependencies:** —
  - **Acceptance criteria:** 3 partners signed; each commits ≥5 NDAs + ≥5 MSAs; each names a counsel + ops point of contact; DPA executed.
- **1.1.2 Baseline capture (current review time + violation sample)**
  - **Deliverable:** Baseline dataset per partner (median manual review minutes; retrospective violation sample)
  - **Owner:** Product
  - **Dependencies:** 1.1.1
  - **Acceptance criteria:** Median review time recorded for ≥10 contracts/partner; ≥20-contract retrospective sample labeled for violations; stored as the success-metric baseline.

#### 1.2 Requirements & taxonomy
- **1.2.1 Lock NDA + MSA clause taxonomy**
  - **Deliverable:** Approved clause taxonomy doc (confidentiality, term/termination, liability cap, indemnification, IP, auto-renewal, governing law, assignment, warranties)
  - **Owner:** Legal Ops
  - **Dependencies:** 1.1.1
  - **Acceptance criteria:** Taxonomy covers ≥10 categories for NDA + MSA; signed off by ≥2 partner counsels; each category has a definition + example.
- **1.2.2 Define grounding & guardrail policy**
  - **Deliverable:** Written policy: "no citation → no output," abstain rules, suggested-only redlines
  - **Owner:** Product
  - **Dependencies:** 1.2.1
  - **Acceptance criteria:** Policy enumerates escalation triggers; specifies confidence thresholds; approved by Legal.

#### 1.3 Environment & infrastructure
- **1.3.1 Provision cloud env, CI/CD, secrets management**
  - **Deliverable:** Dev/staging environments + CI/CD pipeline
  - **Owner:** DevOps
  - **Dependencies:** —
  - **Acceptance criteria:** CI runs on every PR; staging deploy automated; secrets stored in a vault (no plaintext keys).
- **1.3.2 Stand up data stores (object storage, DB, vector store)**
  - **Deliverable:** Provisioned Postgres + object storage + vector store, per-tenant isolation
  - **Owner:** BE
  - **Dependencies:** 1.3.1
  - **Acceptance criteria:** Tenant-isolated schemas verified; vector store returns top-k; encryption at rest enabled.

---

### 2. Document Ingestion & Parsing

#### 2.1 Upload & storage
- **2.1.1 Contract upload service (PDF/DOCX ≤25MB)**
  - **Deliverable:** Upload API + secure storage with virus scan
  - **Owner:** BE
  - **Dependencies:** 1.3.2
  - **Acceptance criteria:** Accepts PDF/DOCX ≤25MB; rejects others with clear error; file encrypted at rest; upload event logged.
- **2.1.2 Language + type detection (English; NDA/MSA)**
  - **Deliverable:** Pre-check module
  - **Owner:** ML
  - **Dependencies:** 2.1.1
  - **Acceptance criteria:** Non-English rejected gracefully; doc-type classifier ≥90% on labeled NDA/MSA set; out-of-scope types flagged.

#### 2.2 Text extraction & OCR
- **2.2.1 Layout-aware text extraction + OCR fallback**
  - **Deliverable:** Parsing pipeline with per-page OCR confidence
  - **Owner:** ML
  - **Dependencies:** 2.1.1
  - **Acceptance criteria:** ≥99% char accuracy on machine-readable test set; scanned docs trigger OCR; low-confidence pages flagged, never silently dropped.

#### 2.3 Clause segmentation
- **2.3.1 Segment contract into clauses with stable IDs + offsets**
  - **Deliverable:** Segmentation service emitting clause IDs + source char offsets
  - **Owner:** ML
  - **Dependencies:** 2.2.1, 1.2.1
  - **Acceptance criteria:** ≥90% boundary F1 on annotated gold set; each clause maps to exact source span for highlighting.

---

### 3. Playbook Builder

#### 3.1 Playbook ingestion & inference
- **3.1.1 Ingest past contracts & infer candidate standards**
  - **Deliverable:** Inference job producing per-category candidate standards + evidence + confidence
  - **Owner:** ML
  - **Dependencies:** 2.3.1, 1.2.1
  - **Acceptance criteria:** Each candidate cites source clauses; thresholds extracted where present; conflicts surfaced for resolution.
- **3.1.2 Playbook data model + versioning**
  - **Deliverable:** Versioned playbook schema (approved language, hard/soft constraints, jurisdiction assumptions)
  - **Owner:** BE
  - **Dependencies:** 1.3.2
  - **Acceptance criteria:** Each rule has immutable version ID; hard vs soft typed; queryable by clause category.

#### 3.2 Playbook review UI
- **3.2.1 Standard accept/edit/reject UI**
  - **Deliverable:** Playbook builder screen
  - **Owner:** FE
  - **Dependencies:** 3.1.1, 3.1.2, 3.3.1
  - **Acceptance criteria:** Counsel can accept/edit/reject each candidate; no standard active without explicit acceptance; published as versioned playbook.

#### 3.3 Playbook UX
- **3.3.1 Playbook builder UX flow & wireframes**
  - **Deliverable:** Figma flows for setup + standard review
  - **Owner:** UX
  - **Dependencies:** 1.2.1
  - **Acceptance criteria:** Flow validated with ≥2 partner counsels; shows evidence per candidate standard; ≤6 steps to publish v1.

---

### 4. Analysis Pipeline (Planner-Executor + RAG)

#### 4.1 Orchestration
- **4.1.1 Planner-executor orchestrator**
  - **Deliverable:** Pipeline: segment → classify → retrieve → score → draft redline
  - **Owner:** ML
  - **Dependencies:** 2.3.1, 4.2.1, 4.3.1
  - **Acceptance criteria:** Processes a contract end-to-end; per-step logging/tracing; resumable on failure; emits structured results.

#### 4.2 Clause classification
- **4.2.1 Clause type classifier (NDA + MSA taxonomy)**
  - **Deliverable:** Classification model/service
  - **Owner:** ML
  - **Dependencies:** 2.3.1, 1.2.1
  - **Acceptance criteria:** ≥85% top-1 accuracy on gold set; low-confidence → "unclassified" not mis-bucketed.

#### 4.3 Retrieval & grounding
- **4.3.1 RAG retrieval over playbook + citation binding**
  - **Deliverable:** Retrieval service returning playbook rule(s) + citation per clause
  - **Owner:** ML
  - **Dependencies:** 3.1.2, 4.2.1
  - **Acceptance criteria:** Every output carries resolvable citation (rule ID + contract span); retrieval precision@k validated on gold set.

#### 4.4 Deviation/risk scoring
- **4.4.1 Deviation detection + severity scoring**
  - **Deliverable:** Scoring module {compliant/deviation, severity, explanation}
  - **Owner:** ML
  - **Dependencies:** 4.3.1, 1.2.2
  - **Acceptance criteria:** High-severity recall ≥0.85; false-positive rate ≤15%; plain-language explanation per flag.

#### 4.5 Redline generation
- **4.5.1 Grounded redline suggestion + verification pass**
  - **Deliverable:** Redline generator producing diffs grounded in approved language
  - **Owner:** ML
  - **Dependencies:** 4.4.1, 3.1.2
  - **Acceptance criteria:** Each high/med deviation yields a redline diff; verification re-checks vs cited rule; redlines marked suggested-only.

#### 4.6 Evaluation harness
- **4.6.1 Gold-set eval harness + release gating**
  - **Deliverable:** Automated eval pipeline + dashboards
  - **Owner:** ML
  - **Dependencies:** 4.2.1, 4.3.1, 4.4.1
  - **Acceptance criteria:** Reports classification/recall/grounding metrics per build; blocks release if grounding <100% or recall regresses below threshold.

---

### 5. Review Experience & Workflow

#### 5.1 Review UI
- **5.1.1 Contract review screen (risk summary + flagged clauses)**
  - **Deliverable:** Review UI with citation, explanation, redline diff per clause
  - **Owner:** FE
  - **Dependencies:** 4.1.1, 5.4.1
  - **Acceptance criteria:** Original ↔ redline ↔ citation visible together; filter by severity/category/status; action in ≤3 clicks.
- **5.1.2 Accept/edit/reject redline controls**
  - **Deliverable:** Interactive redline controls + edit capture
  - **Owner:** FE
  - **Dependencies:** 5.1.1
  - **Acceptance criteria:** Each redline can be accepted/edited/rejected; edits captured for calibration; UI reflects action ≤300ms p95.

#### 5.2 Human approval workflow & RBAC
- **5.2.1 Roles + approval gating (Submitter/Reviewer/Approver/Admin)**
  - **Deliverable:** RBAC + server-enforced approval workflow
  - **Owner:** BE
  - **Dependencies:** 1.3.2, 5.1.2
  - **Acceptance criteria:** Submitter cannot finalize (server-enforced); high-severity flags block "reviewed" unless overridden+logged; every decision logged.

#### 5.3 Export & versioning
- **5.3.1 Redlined DOCX + summary memo export; version history**
  - **Deliverable:** Export service + contract version history
  - **Owner:** BE
  - **Dependencies:** 5.1.2, 5.2.1
  - **Acceptance criteria:** DOCX opens with tracked changes intact; memo PDF generated; each round versioned; records playbook version applied.

#### 5.4 Review UX
- **5.4.1 Review flow UX & wireframes**
  - **Deliverable:** Figma flows for review + escalation
  - **Owner:** UX
  - **Dependencies:** 3.3.1
  - **Acceptance criteria:** Validated with ≥2 partners; escalation path clear; severity hierarchy legible.

---

### 6. Security, Compliance & Audit

#### 6.1 Audit logging
- **6.1.1 Immutable audit log (access + decisions + playbook version)**
  - **Deliverable:** Append-only audit log + export report
  - **Owner:** BE
  - **Dependencies:** 1.3.2, 5.2.1
  - **Acceptance criteria:** Logs upload/parse/flag/decision/export with actor+timestamp; tamper-test passes; exportable audit report.
- **6.1.2 Data retention & deletion controls**
  - **Deliverable:** Configurable retention + hard-delete; no-train guarantee
  - **Owner:** BE
  - **Dependencies:** 1.3.2
  - **Acceptance criteria:** Default 12-mo retention configurable; hard delete verified; contracts/playbooks excluded from model training (documented).

#### 6.2 Disclaimers & security review
- **6.2.1 In-product disclaimers + onboarding acknowledgment**
  - **Deliverable:** Disclaimer copy in UI + exports; acknowledgment gate
  - **Owner:** Legal Ops
  - **Dependencies:** 1.2.2
  - **Acceptance criteria:** "Not legal advice" persistent in review UI + every export; onboarding requires acknowledgment.
- **6.2.2 Pre-launch security review + pen test**
  - **Deliverable:** Security review report + remediation
  - **Owner:** DevOps
  - **Dependencies:** 6.1.1, 6.1.2, 5.2.1
  - **Acceptance criteria:** TLS 1.2+/AES-256 verified; no critical/high findings open; tenant isolation tested.

---

### 7. Commercialization

#### 7.1 Metering & billing
- **7.1.1 Per-contract usage metering + monthly base billing**
  - **Deliverable:** Usage meter + billing integration + usage dashboard
  - **Owner:** BE
  - **Dependencies:** 4.1.1, 5.3.1
  - **Acceptance criteria:** Each completed review increments usage; monthly base fee billed; partner-visible usage dashboard reconciles to logs.
- **7.1.2 Pricing validation with design partners**
  - **Deliverable:** Validated price (per-contract + base) + signed order forms
  - **Owner:** Product
  - **Dependencies:** 7.1.1, 1.1.2
  - **Acceptance criteria:** ≥2 partners confirm pricing fair vs current cost; ≥1 converts to paid order form.

---

### 8. Quality, Launch & Iteration

#### 8.1 QA & UAT
- **8.1.1 End-to-end QA + design-partner UAT**
  - **Deliverable:** Test suite + UAT sign-off
  - **Owner:** QA
  - **Dependencies:** 5.1.2, 5.2.1, 5.3.1
  - **Acceptance criteria:** E2E happy-path + edge (OCR/odd format) covered; ≥3 partners complete UAT; no open sev-1/2 bugs.

#### 8.2 Launch & measurement
- **8.2.1 MVP launch + success-metric instrumentation**
  - **Deliverable:** Production launch + analytics tracking G1/G2
  - **Owner:** Product
  - **Dependencies:** 6.2.2, 7.1.1, 8.1.1
  - **Acceptance criteria:** All partners live in prod; review-time + catch-rate tracked vs 1.1.2 baseline; ≥1 partner shows −50% review time on control set.

---

## Milestones

| # | Milestone | Target week | Tied WBS items | Exit criteria |
|---|-----------|-------------|----------------|---------------|
| **M1** | Partners signed & baselines captured | End W1 | 1.1.1, 1.1.2 | 3 partners signed; baselines stored |
| **M2** | Foundations ready (taxonomy + infra) | End W2 | 1.2.1, 1.3.1, 1.3.2 | Taxonomy approved; envs + data stores live |
| **M3** | Ingestion pipeline working | End W3 | 2.1.1, 2.2.1, 2.3.1 | NDA/MSA parsed + segmented at target accuracy |
| **M4** | Playbook builder usable | End W4 | 3.1.1, 3.2.1, 3.1.2 | Partner publishes a versioned playbook v1 |
| **M5** | Analysis pipeline grounded & gated | End W5 | 4.1.1, 4.3.1, 4.4.1, 4.6.1 | E2E flags + redlines with 100% citations; eval gates pass |
| **M6** | Review experience + approval/RBAC complete | End W6 | 5.1.1, 5.1.2, 5.2.1, 5.3.1 | Counsel reviews, approves, exports redlined DOCX |
| **M7** | Security, audit & billing ready | End W7 | 6.1.1, 6.2.2, 7.1.1 | Security review clean; audit log + metering live |
| **M8** | MVP launch with metrics | End W8 | 8.1.1, 8.2.1, 7.1.2 | 3 partners live; ≥1 shows −50% review time; ≥1 paid |

---

## Out of Scope (MVP)

1. Contract types beyond NDA and MSA (DPA, SOW, employment, leases, financing docs).
2. Autonomous signing, e-signature, or sending contracts to counterparties.
3. Agent-to-counterparty negotiation / automated back-and-forth.
4. Non-English contracts and multi-jurisdiction legal interpretation engine.
5. Native CRM / CLM / e-signature integrations (manual export only).
6. Obligation tracking, renewals calendar, and other CLM lifecycle features.
7. Bulk/batch upload pipelines beyond single-contract processing.
8. Mobile native apps (responsive web only).
9. Self-host / on-prem / VPC deployment option.
10. Custom per-partner ML fine-tuning (shared models + per-tenant playbook only).
11. Public API / third-party developer integrations.
12. SSO/SCIM enterprise identity provisioning (basic auth + roles only for MVP).

---

## Risk & Mitigation

| # | Risk | WBS ref | Likelihood/Impact | Mitigation |
|---|------|---------|-------------------|------------|
| **R1** | Partners can't provide enough clean past contracts to seed playbooks | 1.1.1, 3.1.1 | Med / High | Provide template-based cold-start playbook; lower minimum to 5+5; manual standard entry path. |
| **R2** | No reliable baseline → can't prove −50% / catch-rate | 1.1.2, 8.2.1 | Med / High | Make baseline capture a W1 gate; use timed control set if historical data is weak. |
| **R3** | OCR/format issues on scanned contracts cause silent errors | 2.2.1 | High / High | Per-page confidence; hard-block low-confidence; never silently drop; QA on messy corpus (8.1.1). |
| **R4** | Clause segmentation/classification below target accuracy | 2.3.1, 4.2.1 | Med / High | Gold-set eval (4.6.1); "unclassified" fallback; iterate prompts/few-shot before scope cuts. |
| **R5** | Ungrounded / hallucinated redlines erode trust | 4.3.1, 4.5.1 | Med / Critical | "No citation → no output" guardrail; verification pass; suggested-only framing; eval gating. |
| **R6** | High false-positive flag rate frustrates counsel | 4.4.1 | Med / Med | Calibrate severity with partner labels; downgrade noisy categories to flag-only; track reject rate. |
| **R7** | Non-lawyers finalize risky terms despite workflow | 5.2.1 | Low / High | Server-enforced RBAC; high-severity blocks self-clear; override requires Approver + audit log. |
| **R8** | Security/data-leakage concerns block partner adoption | 6.1.2, 6.2.2 | Med / High | No-train guarantee; tenant isolation; encryption; pre-launch pen test; DPA in 1.1.1. |
| **R9** | DOCX redline export loses fidelity (tracked changes break) | 5.3.1 | Med / Med | Validate tracked-changes in Word early; QA across Word versions; fallback to memo + clause list. |
| **R10** | Scope creep beyond NDA/MSA derails 6–8 week timeline | 1.2.1 | High / High | Enforce Out-of-Scope list; change-control via Product; defer extras to post-MVP iteration. |
| **R11** | LLM latency/cost too high for responsive review | 4.1.1, 7.1.1 | Med / Med | Cache retrieval; batch clause checks; set latency targets (≤120s p95 NDA); monitor cost/contract. |
| **R12** | Pricing not validated → no conversion | 7.1.2 | Med / High | Validate price vs cost-per-review baseline early; secure ≥1 paid order form before launch (M8). |

---

## Appendix A — Generation Prompt

This WBS was generated from the following prompt.

> You are a project planner for an AI product team.
>
> Using the product brief below, create a Work Breakdown Structure (WBS) for delivering the MVP in 6–8 weeks.
>
> **PRODUCT BRIEF**
> 1. Agent: Contract Review Agent for SMB Legal
> 2. Idea: Review inbound vendor/customer contracts against a company playbook; flag risky clauses with explanations and suggested redlines.
> 3. Target: In-house counsel or ops leads at SMBs.
> 4. Scope/MVP: NDA + MSA only; playbook builder from past contracts; clause flagging + redline suggestions in a review view; 3 design partners; commercial motion (per-contract usage pricing + monthly SaaS base fee).
> 5. Architecture: RAG grounding on company playbook; planner-executor pattern (segment contract -> check each clause type); human-in-the-loop redline acceptance.
> 6. Success metrics: -50% review time per contract; increased % of contracts caught with playbook violations.
>
> **OUTPUT REQUIREMENTS**
> A. Provide a WBS with 4 levels max, using numbered items like 1.1.1.
> B. For each WBS leaf task (Level 3 or 4), include:
>    - Task/Activity
>    - Deliverable (what artifact is produced)
>    - Owner role (e.g., ML Engineer, Backend Engineer, Frontend Engineer, Legal Ops, Product, QA, UX)
>    - Dependencies (up to 3 items referencing other WBS numbers)
>    - Acceptance criteria (2–4 measurable checks)
> C. Add a Milestones section after the WBS with 6–8 milestones, each tied to at least one WBS item.
> D. Add an "Out of Scope" section listing 8–12 items explicitly excluded from MVP.
> E. Add a "Risk & Mitigation" section with 8–12 risks tied to WBS items (reference WBS numbers).
> F. Keep it focused on shipping the MVP (not long-term platform R&D).
>
> START NOW. Produce only the WBS, Milestones, Out of Scope, and Risk & Mitigation.
