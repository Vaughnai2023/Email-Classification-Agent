# Inbox Inferno — Email Classification Agent

**An intelligent n8n workflow that classifies inbound customer emails into 6 categories and drafts grounded, documentation-backed replies — with a built-in evaluation system to prove it works.**

Built for the [Inbox Inferno at Nexus Integrations](https://community.n8n.io) community challenge.

---

## What It Does

1. **Receives** an inbound customer email (from, subject, body) via webhook
2. **Classifies** it into one of 6 categories — pricing, support, security, setup, off-topic, or escalate — using a decision-tree prompt with confidence scoring
3. **Injects** only the relevant documentation section for that category (not the entire 310-line doc)
4. **Drafts** a grounded reply that never hallucinates — citing only what's in the docs, and flagging itself when it can't answer

## Architecture

```
┌───────────┐     ┌─────────────┐     ┌─────────────────┐     ┌──────────────┐
│  Webhook  │────▶│  Normalize  │────▶│   Classifier     │────▶│  Confidence  │
│  Trigger  │     │   Input     │     │  (gpt-4o-mini)   │     │    Gate      │
└───────────┘     └─────────────┘     └─────────────────┘     └──────┬───────┘
                                                                      │
┌───────────┐                                              ┌──────────▼───────┐
│   Eval    │─────────────────────────────────────────────▶│ Category Router  │
│  Trigger  │                                              │   (6 outputs)    │
└───────────┘                                              └──────────┬───────┘
                                                                      │
                                                           ┌──────────▼───────┐
                                                           │   Doc Injector   │
                                                           │ (scoped lookup)  │
                                                           └──────────┬───────┘
                                                                      │
                                                           ┌──────────▼───────┐
                                                           │  Reply Drafter   │
                                                           │    (gpt-4o)      │
                                                           └──────────┬───────┘
                                                                      │
                                                           ┌──────────▼───────┐
                                                           │   Source Gate    │
                                                           └────┬────────┬───┘
                                                                │        │
                                                    ┌───────────▼┐  ┌───▼──────────┐
                                                    │  Respond   │  │  Evaluation  │
                                                    │  to Client │  │  Pipeline    │
                                                    └────────────┘  └──────────────┘
```

**Low confidence?** The confidence gate auto-escalates rather than guessing wrong. Safety over speed.

---

## Screenshots

> **Coming soon** — screenshots will be added after the workflow is deployed and evaluated.

| | |
|---|---|
| ![Workflow Overview](screenshots/workflow-overview.png) | ![Eval Results](screenshots/eval-results.png) |
| *Full n8n workflow canvas* | *Evaluation run — pass rates across 20 test cases* |
| ![Classification Example](screenshots/classification-example.png) | |
| *Example: email classified and reply drafted* | |

---

## Who Is This For?

- **Businesses drowning in support email** — See how AI can triage and draft responses without hallucinating or losing angry customers
- **n8n builders** — Learn a production-grade pattern for LLM-powered email agents with built-in evaluation
- **AI automation consultants** — Fork this as a starting point for client email automation projects
- **Anyone evaluating AI reliability** — The 3-layer eval system (exact match + LLM judge + self-audit) is a reusable pattern for proving AI agents work

---

## Key Decisions

| What most builders do | What we do instead | Why |
|---|---|---|
| Single "classify + reply" LLM call | Separated classification → generation pipeline | Isolating classification lets us optimize accuracy independently from reply quality |
| Full doc dump in every prompt | Category-scoped doc injection (~1-2k tokens vs ~6k) | Cheaper, faster, fewer distractions for the LLM |
| Escalation as fallback | Escalation checked FIRST + confidence circuit breaker | The scoring function penalizes misclassification as total failure — escalating safely is always better than guessing wrong |
| Same model for everything | gpt-4o-mini for classification, gpt-4o for reply | Classification is a routing decision (small/fast model). Reply generation needs nuance (larger model) |
| Basic pass/fail eval | Three-layer eval: exact match + LLM judge + self-audit flag | Proves both classification accuracy AND reply groundedness |

---

## Repo Structure

```
inbox-inferno/
├── nexus-docs.md          ← Nexus Integrations product documentation (source of truth for the agent)
├── build-plan.md          ← Detailed architecture plan — node-by-node workflow design
├── test-emails.json       ← 20 test cases with expected categories and evaluation notes
├── screenshots/           ← Workflow and evaluation screenshots (to be added)
├── .gitignore
└── README.md
```

---

## How to Use This

### Prerequisites

- [n8n](https://n8n.io) instance (self-hosted or cloud)
- OpenAI API key (gpt-4o and gpt-4o-mini access)

### Step 1: Study the Architecture

Read [`build-plan.md`](build-plan.md) for the complete node-by-node workflow design, prompt strategies, and evaluation wiring.

### Step 2: Review the Documentation

[`nexus-docs.md`](nexus-docs.md) is the ground truth the agent uses. It covers pricing, setup, security, support, and contact information for the fictional Nexus Integrations company.

### Step 3: Explore the Test Dataset

[`test-emails.json`](test-emails.json) contains 20 carefully crafted test emails spanning all 6 categories, including edge cases like multi-category emails, vague requests, and angry customers. Each includes an `expected_category` and detailed notes.

### Step 4: Build the Workflow

Follow the build sequence in `build-plan.md` to construct the workflow in your n8n instance. The plan includes all prompts, node configurations, and wiring details.

### Step 5: Evaluate

Load the 20 test cases into an n8n Data Table, run the evaluation trigger, and verify:
- **Categorization accuracy** >= 90%
- **LLM Judge score** >= 85%
- **All escalation cases** correctly routed

---

## Tech Stack

- **Workflow engine**: [n8n](https://n8n.io) — open-source workflow automation
- **Classification**: OpenAI gpt-4o-mini (temp: 0) with structured output parsing
- **Reply generation**: OpenAI gpt-4o (temp: 0.3) with grounding enforcement
- **Evaluation**: n8n built-in eval framework + custom LLM judge
- **Documentation**: Custom Nexus Integrations product docs (310 lines, 6 sections)

---

## Evaluation System

The agent proves its own reliability through a **3-layer evaluation pipeline**:

| Layer | What It Tests | Method |
|-------|--------------|--------|
| **Categorization** | Does the agent pick the right category? | Exact match: `agent_category` vs `expected_category` |
| **LLM Judge** | Is the reply grounded in docs? Is escalation appropriate? | GPT-4o scores 0 or 1 based on a rubric mirroring the challenge scoring function |
| **Self-Audit** | Did the agent go beyond the documentation? | The reply drafter sets a `contains_info_not_in_docs` flag on itself |

A response only scores full marks when the category is correct **AND** the reply is grounded **AND** escalation is appropriate.

---

## Want This Built for Your Business?

If your team is drowning in customer emails and you want an AI agent that actually works — not one that hallucinates pricing or loses angry customers — let's talk.

I build reliable AI automation workflows with n8n for businesses that need their email triage, classification, and response drafting to be accurate and auditable.

**Get in touch:**
- [LinkedIn — Vaughn Botha](https://www.linkedin.com/in/vaughnbotha/)
- [vaughnai2023@gmail.com](mailto:vaughnai2023@gmail.com)

---

Built by **Vaughn Botha** · March 2026
