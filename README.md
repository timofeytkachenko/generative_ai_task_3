# Generative AI — Homework 3

This repository contains a set of **prompt-engineering and product-ideation deliverables** produced for the Generative AI course. It documents an end-to-end flow from broad LLM/agentic startup ideation down to a single chosen idea expanded into a **PRD** and a **delivery plan (WBS)** — plus a separate ML prompt-engineering exercise.

Every document is self-contained and includes, in an appendix, the exact prompt used to generate it (for reproducibility and grading transparency).

---

## Project Structure

```text
generative_ai_hw_3/
├── README.md                        # This file — index and guide to all documents
├── IDEAS_README.md                  # 10 LLM/agentic startup ideas + generation prompt
├── PRD_CONTRACT_REVIEW_AGENT.md     # Full PRD for the chosen idea (#3) + generation prompt
├── WBS_CONTRACT_REVIEW_AGENT.md     # 6–8 week MVP delivery plan (WBS) + generation prompt
└── PROMPT_torch_titanic.md          # Standalone ML prompt: improve a PyTorch Titanic baseline
```

---

## Document Index

### `IDEAS_README.md`
**Startup ideation deliverable.** Ten distinct B2B startup ideas centered on LLM + agentic systems, aligned to 2026 trends. Each idea follows a fixed 9-field structure (idea, target customer, pain, solution, why-now, how LLM/agents are used, MVP scope, difficulty, success metric) and ends with a comparison table. Includes the original generation prompt in a blockquote.

- **Role in the flow:** the funnel's top — the source of candidate ideas.
- **Outcome:** idea **#3, Contract Review Agent for SMB Legal**, was selected as the easiest/most viable to realize.

### `PRD_CONTRACT_REVIEW_AGENT.md`
**Product Requirements Document** for the selected idea (#3). A decision-ready PRD written for engineering, legal/compliance, and design, covering 13 sections: executive summary, problem statement, goals/non-goals, personas, user stories & flows, MVP scope, functional requirements, non-functional requirements, playbook builder spec, risk & compliance, metrics plan, rollout plan, and open questions. Generation prompt is included as **Appendix A**.

- **Role in the flow:** turns one idea into a concrete, buildable product definition.
- **Key concepts:** RAG grounding on a company playbook, planner-executor pipeline, human-in-the-loop redline approval; NDA + MSA scope only.

### `WBS_CONTRACT_REVIEW_AGENT.md`
**Work Breakdown Structure** for delivering the PRD's MVP in 6–8 weeks. Contains an 8-workstream WBS (4 levels deep) with per-task deliverable/owner/dependencies/acceptance criteria, 8 week-mapped milestones, a 12-item out-of-scope list, and a 12-item risk & mitigation table referencing WBS numbers. Generation prompt is included as **Appendix A**.

- **Role in the flow:** turns the PRD into an executable, time-boxed delivery plan.
- **Reads alongside:** `PRD_CONTRACT_REVIEW_AGENT.md` (requirements ↔ tasks trace to each other).

### `PROMPT_torch_titanic.md`
**Standalone ML prompt-engineering exercise** (independent of the contract-review thread). A reproducible prompt instructing an agent to improve a PyTorch Titanic MLP baseline (`r_d_Lesson_2_Torch_Titanic.ipynb`) by +5–10% relative test quality through **architecture + training-procedure changes only** (feature set and held-out test split are frozen for an honest comparison).

- **Role in the flow:** separate deliverable demonstrating prompt design for a constrained ML improvement task.

---

## How the Documents Relate

```text
IDEAS_README.md            (10 ideas)
        │
        │  select idea #3 (Contract Review Agent)
        ▼
PRD_CONTRACT_REVIEW_AGENT.md   (what to build + why)
        │
        │  decompose into deliverable tasks
        ▼
WBS_CONTRACT_REVIEW_AGENT.md   (how/when to build it)

PROMPT_torch_titanic.md        (independent ML prompt exercise)
```

---

## Conventions

- All files are **Markdown** and intended to be read in any Markdown viewer.
- Each generated document ends with an **Appendix A — Generation Prompt** containing the verbatim prompt used to produce it, so results are reproducible and auditable.
- The contract-review documents share terminology (playbook, RAG grounding, planner-executor, human-in-the-loop) and cross-reference each other (PRD requirements ↔ WBS tasks).
