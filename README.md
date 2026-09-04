# CONTRIBUTIONS.md
## Team C-5 · Problem A: Health-Insurance Claim First Response
## PE6201 Emerging AI Technologies · T1 AY2026–27

---

## Team Members & Task Division

| # | Name | Role | Deliverables | Report Section | Live Model | Tier |
|---|------|------|-------------|----------------|------------|------|
| 1 | SOOD VEDANSH | Lead Coder | D1, D2(c), D3(a), D5(a), D7 | — | `mistralai/mistral-7b-instruct` | Cheap |
| 2 | YANG YICHEN | Support Coder | Code review, D0(a) support, D2(c) dependency design, D3(a)/D7 stop-condition sizing | Sections 5 & 6 | `google/gemma-3-4b-it` | Cheap |
| 3 | ZHANG QIZHI | Conceptual Lead | D0(a), D0(b), D0(c), D2(a) | Section 1 | `mistralai/mistral-small-3.2-24b-instruct` | Mid |
| 4 | CHEN BAIYI | Tool Design Lead | D2(b) — descriptors, poka-yoke, v1 vs v2 rewrite | Section 2 | v1 pass on `mistralai/mistral-7b-instruct` (same as Member 1) | Cheap |
| 5 | QIAN RUOQI | Eval & Guardrail Lead | D3(b), D4 | Section 3 | `meta-llama/llama-3.1-8b-instruct` | Cheap |
| 6 | FANG HIOIENG | Cost Analysis Lead | D6 — cost model, sensitivity table, break-even, four-lever ledger | Sections 3 & 4 | `google/gemini-2.0-flash-lite-001` | Mid |

---

## Model Assignments

| Member | Model (OpenRouter string) | Tier | Price (in/out per 1M tokens) | Family |
|--------|--------------------------|------|------------------------------|--------|
| SOOD VEDANSH | `mistralai/mistral-7b-instruct` | Cheap | $0.10 / $0.40 | Mistral |
| YANG YICHEN | `google/gemma-3-4b-it` | Cheap | $0.10 / $0.40 | Google |
| ZHANG QIZHI | `mistralai/mistral-small-3.2-24b-instruct` | Mid | $1.00 / $5.00 | Mistral |
| QIAN RUOQI | `meta-llama/llama-3.1-8b-instruct` | Cheap | $0.10 / $0.40 | Meta |
| FANG HIOIENG | `google/gemini-2.0-flash-lite-001` | Mid | $1.00 / $5.00 | Google |
| CHEN BAIYI | v1 pass on `mistralai/mistral-7b-instruct` | Cheap | $0.10 / $0.40 | Mistral (held fixed for descriptor comparison) |

> Models span two price tiers (cheap + mid). No two members share the same model family except where one is the v1 pass — which requires the same model as Member 1 by design.

---

## Estimated Live Spend per Member

| Member | Runs | Tier | Est. Cost |
|--------|------|------|-----------|
| SOOD VEDANSH | 56 (v2) | Cheap | ~US$0.27 |
| YANG YICHEN | 56 (v2) | Cheap | ~US$0.27 |
| ZHANG QIZHI | 56 (v2) | Mid | ~US$2.76 |
| QIAN RUOQI | 56 (v2) | Cheap | ~US$0.27 |
| FANG HIOIENG | 56 (v2) | Mid | ~US$2.76 |
| CHEN BAIYI | 56 (v1) | Cheap | ~US$0.27 |

> All within the US$3/member budget ceiling. Debug on scripted backend only — live tokens are for the final battery.

---

## What Every Member Must Do

- Write ~7 evaluation cases each (40 total — Member 5 coordinates)
- Run one live model on their own OpenRouter key
- Speak in the 5-minute recorded demo (must show a negative case)
- Complete peer rating by Wed 16 Sep 2026, 23:59 SGT

---

## Key Dates

| Date | Milestone |
|------|-----------|
| Before first agent commit | D0(c) committed by ZHANG QIZHI |
| Wed 2 Sep, midday | Fixture data + scaffold on NTULearn |
| **Fri 4 Sep, 23:59 SGT** | TEAM_DECLARATION.docx due |
| **Sun 13 Sep, 23:59 SGT** | A2 due |
| Wed 16 Sep, 23:59 SGT | Peer rating due |
