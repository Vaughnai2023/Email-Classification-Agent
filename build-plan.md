# Inbox Inferno — n8n Workflow Build Plan

## Context

Building an n8n workflow for the "Inbox Inferno at Nexus Integrations" community challenge. The challenge requires two parts: (1) an email agent that classifies inbound emails and drafts grounded replies using Nexus documentation, and (2) an evaluation system using n8n's built-in eval framework to prove the agent works reliably. The deadline passed (22/03/2026) but we're building this as a portfolio piece / late submission. We created our own Nexus docs and test dataset since the resource pack is no longer available.

**Core insight driving the design:** The scoring function rewards correct escalation equally to correct answers, and penalizes wrong classification as a total failure (0). This means our architecture should **over-invest in classification accuracy and escalation safety** rather than reply eloquence.

---

## Architecture Overview

**Two-phase pipeline with confidence circuit breaker:**

```
Webhook ──┐
           ├──→ Normalize Input → Classifier (LLM #1) → Confidence Gate → Category-Scoped Docs → Reply Drafter (LLM #2) → Source Gate → Output
EvalTrigger┘
```

- **2 LLM calls max** per email (classify + draft). Most builders will use 3+.
- **Category-scoped doc injection** — only relevant doc section enters the reply prompt, not all 310 lines.
- **Confidence gate** — low-confidence classifications auto-escalate instead of guessing wrong.
- **Escalation checked FIRST** in classification (before any other category).
- **Source gate** — routes webhook runs to Respond to Webhook, eval runs to Evaluation metrics.

---

## What We're Building (Node-by-Node)

### Entry Points (dual trigger — no Merge node)

| # | Node | Type | Purpose |
|---|---|---|---|
| 1 | Webhook | `webhook` | Production POST endpoint receiving `{from, subject, body}` |
| 2 | Evaluation Trigger | `evaluationTrigger` | Reads test cases from Data Table, sends one-at-a-time |

Both connect **directly** to the Normalize Input node. No Merge node — n8n runs the downstream chain for whichever trigger fires. A Merge node would deadlock waiting for both inputs.

### Classification Phase

| # | Node | Type | Purpose |
|---|---|---|---|
| 3 | Set: Normalize Input | `set` | Standardize field names from both entry points. Sets `source` flag ("webhook" or "eval"). Preserves `expected_category` from eval trigger data for downstream comparison. |
| 4 | Classifier | `chainLlm` | Decision-tree classification prompt. Model: `gpt-4o-mini` (temp: 0). Structured Output Parser enforces: `{reasoning, category, confidence, escalation_signals}` |
| 5 | If: Confidence Gate | `if` | Routes `low` confidence → force escalate. `high`/`medium` → continue |
| 6 | Set: Force Escalate | `set` | Overrides category to `escalate` when confidence is low |

**Classifier prompt strategy — ordered decision tree:**
1. FIRST: Check for escalation signals (anger, threats, data loss, legal, vague/ambiguous, no actionable context)
2. SECOND: Check if off-topic (not about Nexus at all — wrong company, job applications, vendor pitches)
3. THIRD: Determine primary intent:
   - Asking *whether they can access* a feature or *what plan they need* → `pricing`
   - Cost/plans/upgrade/billing/plan comparison → `pricing`
   - Compliance/certifications/encryption/data residency/security questionnaires → `security`
   - First-time setup/configuration/connecting/how to configure a feature → `setup`
   - Broken/errors/troubleshooting existing integration/unexpected behavior → `support`

**Edge case rules in prompt:**
- "When a customer's primary question is *how to configure or use* a feature (even if their plan limits it), classify as the functional category (setup/support). When the customer's primary question is *whether they can access* a feature or *what plan they need for it*, classify as pricing."
- "If the email contains no actionable context (vague, references unknown prior conversations, no specifics), classify as escalate."

### Documentation Routing

| # | Node | Type | Purpose |
|---|---|---|---|
| 7 | Switch: Category Router | `switch` | 6 named outputs + **default fallback → escalate** |
| 8 | Code: Doc Injector | `code` | Single Code node with lookup object keyed by category. Returns only the relevant doc section(s). |

The Switch node includes a **default/fallback output** that routes to escalation if the classifier returns an unexpected category value.

**Doc section mapping (injected by Code node):**
- `pricing` → Pricing Plans (all 3 tiers with feature tables) + Integration list summary
- `support` → Troubleshooting table + Data Mapping + Sync intervals + Integration auth methods + Plan-specific limits
- `security` → Security & Compliance (full section) + Contact info (security@, legal@)
- `setup` → Setup guides + Salesforce walkthrough + Integration auth methods + Sync interval configuration + Plan-specific interval limits
- `off-topic` → Contact info only (for redirection) + brief company description
- `escalate` → Support contacts + Escalation path + General company context

Cross-cutting info (sync intervals, plan feature comparison) is included in **both** `setup` and `support` scopes since both categories encounter plan-limited questions.

### Reply Generation

| # | Node | Type | Purpose |
|---|---|---|---|
| 9 | Reply Drafter | `chainLlm` | Generates grounded reply. Model: `gpt-4o` (temp: 0.3). Structured Output Parser: `{draft_reply, citations_used, contains_info_not_in_docs}` |
| 10 | Set: Format Output | `set` | Constructs final `{category, draft_reply}` |

**Reply drafter prompt rules:**
- ONLY use provided documentation. Never invent features, prices, or timelines.
- When docs don't cover the question → say so explicitly, direct to appropriate contact.
- Security questionnaires → always redirect to security@nexusintegrations.com
- Enterprise pricing → never quote a number, direct to sales@
- Reply in same language as the email
- Address sender by name when available
- For escalation category → acknowledge the customer's concern, assure them a team member will follow up, provide relevant contact/escalation info
- For off-topic → politely clarify they've reached Nexus Integrations (a software integration platform), redirect appropriately
- `contains_info_not_in_docs` self-audit: if true, the reply drafter flags itself — this becomes an input to the eval judge

### Output — Source Gate (conditional routing)

| # | Node | Type | Purpose |
|---|---|---|---|
| 11 | If: Source Gate | `if` | Checks `source` field — routes "webhook" to Respond to Webhook, "eval" to Evaluation nodes |
| 12 | Respond to Webhook | `respondToWebhook` | Returns `{category, draft_reply}` for production webhook calls only |
| 13 | Evaluation: Categorization | `evaluation` | Built-in exact match: `category` vs `expected_category` (0/1) |
| 14 | LLM Judge | `chainLlm` | Custom 0/1 scoring: grounding + escalation appropriateness |
| 15 | Evaluation: Judge Score | `evaluation` | Custom metric from judge score |

The Source Gate prevents `respondToWebhook` from firing during eval runs (which would throw an error since there's no active webhook request).

**Evaluation wiring detail:**
- Normalize Input preserves `expected_category` from the eval trigger's test data
- Evaluation: Categorization compares `{{ $json.category }}` (agent output) vs `{{ $json.expected_category }}` (test data)
- LLM Judge receives: agent's `category`, `draft_reply`, `expected_category`, `contains_info_not_in_docs` flag, and the relevant doc section for grounding verification
- Judge Score passes the judge's 0/1 output to the final Evaluation metric node

**LLM Judge prompt evaluates:**
- Is the category correct? (compared to expected_category)
- Is the reply grounded in documentation only? (no hallucinations)
- If escalation was needed, did the agent escalate properly?
- Does the self-audit flag (`contains_info_not_in_docs`) indicate the reply went beyond docs?
- Score 1 = (correct category AND grounded reply) OR (proper escalation when warranted). Score 0 = anything else.

---

## Prompts

All three prompts will be written during the build session and tuned iteratively against the 20 test cases. The key design decisions are locked in above — the prompts implement them.

| Prompt | Model | Key Design Principle |
|---|---|---|
| Classifier | gpt-4o-mini (temp: 0) | Ordered decision tree: escalate → off-topic → intent. Structured output with confidence level. |
| Reply Drafter | gpt-4o (temp: 0.3) | Grounding-enforced with self-audit. Category-aware tone. Never invent. |
| LLM Judge | gpt-4o (temp: 0) | Mirror challenge scoring rubric exactly. Output reasoning + 0/1 score. |

---

## Files

| File | Purpose |
|---|---|
| `nexus-docs.md` | Source of truth — chunked by category inside the Code: Doc Injector node |
| `test-emails.json` | 20 test cases — load into n8n Data Table for Evaluation Trigger |
| `workflow.json` | The exported n8n workflow (created during build) |

---

## Differentiation from Other Builders

| What most will do | What we do instead |
|---|---|
| Single "classify + reply" LLM call | Separated classification → generation pipeline |
| Full doc dump in every call | Category-scoped doc injection (~1-2k tokens vs ~6k) |
| Escalation as fallback | Escalation as FIRST check + confidence circuit breaker |
| Same model for everything | gpt-4o-mini for classification, gpt-4o for reply |
| Basic pass/fail eval | Three-layer eval: Categorization + LLM Judge + self-audit field |
| Hope-based accuracy | Deterministic confidence gateway |
| 6 Set nodes for doc routing | Single Code node with category lookup (cleaner, maintainable) |

---

## Build Sequence

1. **Create n8n Data Table** with the 20 test emails (id, from, subject, body, expected_category)
2. **Skeleton:** Webhook + Evaluation Trigger → Normalize Input (with source flag + expected_category passthrough)
3. **Classifier chain:** Basic LLM Chain + gpt-4o-mini + Structured Output Parser with decision-tree prompt
4. **Confidence gate:** If node + Force Escalate Set node
5. **Category router:** Switch node (with default fallback) → Code: Doc Injector node
6. **Reply drafter:** Basic LLM Chain + gpt-4o + Structured Output Parser
7. **Output formatting:** Set node → Source Gate (If node)
8. **Webhook path:** Source Gate true → Respond to Webhook
9. **Eval path:** Source Gate false → Evaluation: Categorization → LLM Judge → Evaluation: Judge Score
10. **Test & iterate:** Run eval against 20 cases, tune prompts for any failures
11. **Activate** production webhook

---

## Verification

1. **Unit test via curl**: POST a sample email → verify `{category, draft_reply}` returns correctly
2. **Run n8n Evaluation**: Trigger against all 20 test cases → verify Categorization metric ≥ 0.9 and custom judge score ≥ 0.85
3. **Edge case manual tests**: Send 3-5 unseen emails covering: multi-category, unknown integration, competitor mention, non-English, gibberish
4. **Token audit**: Check that classification calls use ~500-800 tokens and reply calls use ~1500-2500 tokens (not 6000+)
5. **Escalation safety check**: Verify that vague emails, angry emails, and low-confidence classifications all route to escalate
6. **Source gate check**: Confirm eval runs don't trigger Respond to Webhook errors
