# CONTRIBUTIONS.md

## Team C-5 · Problem A: Health-Insurance Claim First Response
### PE6201 Emerging AI Technologies · T1 AY2026–27

---

## Team Members & Contributions

| # | Name | Role | Main Contributions | Report Section | Live Model / Experiment |
|---|------|------|--------------------|----------------|-------------------------|
| 1 | SOOD VEDANSH | Lead Coder & V2 Integration Lead | D1 agent loop, D2(c) parallel tool execution, D3(a) code-level guardrails, V2 system integration, live evaluation harness, D5 live battery, D7 failure reproduction | Sections 3 & 5 support | `openai/gpt-oss-20b` |
| 2 | YANG YICHEN | System Reliability & Review | Code review, D0(a) support, D2(c) dependency design, D3(a) stop-condition and budget review, D7 reliability analysis, deployment limitations and alternative design discussion | Sections 5 & 6 | `google/gemini-2.5-flash` |
| 3 | ZHANG QIZHI | Conceptual & Agent Design Lead | D0(a) autonomy-ladder placement, D0(b) workflow/agentic analysis, D0(c) pre-commit design, D2(a) tool selection and architecture rationale | Section 1 | `qwen/qwen-2.5-72b-instruct` |
| 4 | CHEN BAIYI | Tool Design & V1 Baseline Lead | D2(b) tool descriptors, poka-yoke design, V1 tool-interface implementation, V1 baseline evaluation, V1→V2 descriptor/interface comparison | Section 2 | V1 baseline on `openai/gpt-oss-20b` |
| 5 | QIAN RUOQI | Evaluation & Guardrail Lead | D3(b) guardrail test design, hostile-text cases, D4 evaluation-set coordination, negative-case analysis, cross-model result consolidation | Section 3 | `z-ai/glm-4.5-air` |
| 6 | FANG HIOIENG | Cost Analysis Lead | D6 three-layer cost model, model-cost comparison, sensitivity analysis, break-even analysis, cost-lever ledger and deployment economics | Section 4 | `deepseek/deepseek-chat-v3.1` |

---

## Model Assignments

The V2 live battery evaluated five model families. GPT-OSS-20B was also used for the V1 baseline to support a controlled V1→V2 comparison.

| Member | Model (OpenRouter string) | Tier | Family | Final V2 Pass Rate |
|--------|---------------------------|------|--------|-------------------:|
| SOOD VEDANSH | `openai/gpt-oss-20b` | Mid | OpenAI | 54/60 — 90.00% |
| ZHANG QIZHI | `qwen/qwen-2.5-72b-instruct` | Mid | Qwen | 53/60 — 88.33% |
| YANG YICHEN | `google/gemini-2.5-flash` | Mid | Google | 51/60 — 85.00% |
| FANG HIOIENG | `deepseek/deepseek-chat-v3.1` | Mid | DeepSeek | 51/60 — 85.00% |
| QIAN RUOQI | `z-ai/glm-4.5-air` | Cheap | GLM | 49/60 — 81.67% |
| CHEN BAIYI | V1: `openai/gpt-oss-20b` | Mid | OpenAI | V1 controlled baseline |

> The live battery spans two price tiers (cheap and mid) and five different model families. The same GPT-OSS-20B model was retained between V1 and V2 for the controlled system comparison.

---

## Shared Team Contributions

All team members contributed to the final evaluation and presentation process, including:

- Designing and reviewing evaluation cases for the shared claim battery.
- Running assigned live-model experiments using OpenRouter.
- Reviewing model failures and negative-case behaviour.
- Contributing to the final report and evidence consolidation.
- Participating in the 5-minute recorded demonstration.

---

## Key Technical Contributions

### V1 → V2 Tool-Interface Refinement

The main V2 interface change added `required_document` directly to the authoritative `check_coverage` result, alongside `procedure_code`, `coverage_status`, preauthorisation requirements and exclusion information.

Two key poka-yoke measures were used:

1. **Explicit document requirement** — `required_document` makes procedure-specific document requirements directly observable instead of requiring the model to infer them.
2. **Gated irreversible action** — `issue_decision_letter` remains application-controlled and is not exposed to the live model, preventing direct model execution of the write action.

The V1→V2 comparison used GPT-OSS-20B on the same 42 unique cases. The observed improvement was accompanied by increased token usage, and the team therefore treats V2 as a combined system/interface intervention rather than attributing the improvement solely to descriptor wording.

### Evaluation Protocol

The final evaluation followed the assignment's trial-aware policy:

- Ordinary case: **1 trial**
- Negative case: **3 total trials**
- 42 unique cases
- 33 ordinary cases
- 9 negative cases
- **60 total trials per model**

The five V2 models showed different performance under the same system configuration, demonstrating that prompt and tool-interface refinements did not produce identical effects across model families.

### Guardrails and Failure Analysis

The system uses deterministic code-level controls including:

- `MAX_TURNS = 8`
- `TOKEN_BUDGET = 60,000`
- Duplicate-action prevention
- Explicit autonomy setting
- Gated `issue_decision_letter`
- Hostile-narrative detection

The team also reproduced two failures by removing individual protections and restoring them:

1. Removing action de-duplication caused repeated tool calls until the turn cap.
2. Removing `required_document` from `check_coverage` caused an incorrect approval for a missing-document case.

These experiments separated loop/control failures from tool-interface failures and identified the appropriate layer for each fix.

---

## Final Outcome

The final V2 system was evaluated across five live model families. GPT-OSS-20B achieved the highest observed pass rate at **54/60 trials (90.0%)** and was selected as the deployment candidate for the coursework cost analysis.

The team does not recommend fully autonomous claim approval. Human confirmation remains required for the gated write action, and additional deterministic evidence, amount validation, audit logging, usage limits and unseen adversarial testing would be required before production deployment.
