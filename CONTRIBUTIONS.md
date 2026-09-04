# CONTRIBUTIONS.md
## Team C-5 · Problem A: Health-Insurance Claim First Response
## PE6201 Emerging AI Technologies · T1 AY2026–27

---

## Team Members

| # | Name | Role |
|---|------|------|
| 1 | SOOD VEDANSH | Lead Coder |
| 2 | YANG YICHEN | Support Coder |
| 3 | ZHANG QIZHI | Conceptual Lead |
| 4 | CHEN BAIYI | Tool Design & Descriptor Lead |
| 5 | QIAN RUOQI | Evaluation & Guardrail Lead |
| 6 | FANG HIOIENG | Cost Analysis Lead |

---

## Task Division

### SOOD VEDANSH — Lead Coder

**Deliverables**

- D1 — ReAct loop implementation (single-agent, one control loop: Thought → Action → Observation → Final)
- D2(c) — Multi-tool-call parsing and execution in a single turn; dependency rule; sequential vs. parallel measurement
  - Corrected figures: 8 sequential turns → 4 parallel turns; 20,800 → 9,600 input tokens (54% saving)
- D3(a) — Code-layer guardrails: step cap, budget ceiling, action de-duplication, autonomy gate (placed in front of `issue_decision_letter`, not the whole agent)
- D5(a) — Scripted backend (`BACKEND = "scripted"` as default); must reproduce end-to-end with no network and no key
- D7 — Technical reproduction of both failures, built as deletions from working agent; both failures and before/after tables run on scripted backend

**Live Model**
- Model A — cheap tier
- Runs v2 (final) prompt
- ~56 runs, ~US$0.27

**Report Sections**
- None (produces outputs all others depend on)

**Dependencies**
- None — start after D0(c) is committed by Zhang Qizhi

---

### YANG YICHEN — Support Coder

**Deliverables**

- Code cross-checks and review across D1, D2(c), D3(a)
- Supports D0(a) — ladder/rung selection and architecture decisions (single-agent ReAct vs. alternatives)
- Supports D2(c) — dependency rule design; which tools may be called in parallel vs. must run sequentially
- Supports D3(a)/D7 — step-cap sizing from evidence (set from median + worst-case turn distribution, not a round number)
- Draft Report Section 5 — The two failures (loop failure + second failure: what layer, why not the other two layers)
- Draft Report Section 6 — What we would not deploy (limits found + one argued paragraph on the multi-agent architecture not built: what it would have caught, what it would have cost, why we stayed single-agent)

**Live Model**
- Model B — mid tier, different model family from Model A
- Runs v2 (final) prompt
- ~56 runs, ~US$2.76

**Report Sections**
- Section 5 (Two failures — with Member 1)
- Section 6 (What we would not deploy)

**Dependencies**
- Works alongside Member 1 throughout

---

### ZHANG QIZHI — Conceptual Lead

**Deliverables**

- D0(a) — Place problem on Class 4's 7-rung ladder; defend rung 7; explain what rungs 1–6 would and would not have delivered; name the first irreversible action (`issue_decision_letter`) as the governance cliff
- D0(b) — Both Capsule 1 tests answered:
  - Ground-truth test: name the systems of record that can contradict the model within seconds (policy row, preauthorisation table, hospital panel status)
  - Reliability arithmetic: s = P^(1/T) using actual measured P (from D4) and T (from D7); show what cutting turns vs. raising step quality each predicts
- D0(c) — Five numbered "what good looks like" statements, **committed to the repository before the first agent code commit** (commit timestamp will be checked)
- D2(a) — Tool set selection and 3-question justification table (fails without it? confusable? cost when unused?); document at least one tool considered and not added, or one tool removed with the observation that removed it

> **Note:** Tool names in Appendix A are suggestions only. Coordinate with Member 1 on final tool names and signatures before Member 4 writes descriptors.

**Live Model**
- Model C — mid tier, different family from Models A and B
- Runs v2 (final) prompt
- ~56 runs, ~US$2.76

**Report Sections**
- Section 1 — Why an agent (D0) + tool set justification (D2a) · ~400 words

**Dependencies**
- D0(c) must be committed **before** Member 1 writes any agent code
- D2(a) tool selection must be agreed with Member 1 and Member 4 before Member 4 writes descriptors

---

### CHEN BAIYI — Tool Design & Descriptor Lead

**Deliverables**

- D2(b) — Six-field descriptor contract for every tool shipped:
  - Fields: NAME + SIGNATURE, WHAT, INPUT, RETURNS (with size bound), FAILS WHEN, IRREVERSIBLE (Yes/No + gate named)
  - No exceptions — every tool must have all six fields
- Poka-yoke identification — at least 2 per tool; for each, state what it makes **impossible** (not merely discouraged)
- Descriptor rewrite — v1 and v2 for one tool:
  - Run v1 on **one model only** (cheap tier — same model as Member 1, so the model is held fixed and the difference is attributable to the descriptor alone)
  - Report for both v1 and v2: tokens returned per call, evaluation pass rate, guardrail cases passed
  - If v2 is not smaller or not safer, say so — an honest null result scores better than an unmeasured one

> **This is the v1 pass (~56 runs, ~US$0.27).** Member 4 owns this run. Coordinate with Member 1 on which cheap-tier model to use so both run the identical model.

**Live Model**
- Owns the v1 descriptor pass — runs v1 prompt on cheap tier model (same as Member 1's model)
- ~56 runs, ~US$0.27

**Report Sections**
- Section 2 — The tool layer · ~450 words (with input on descriptor rewrite measurement from Member 1's token counts)

**Dependencies**
- Needs final tool signatures agreed with Members 1 and 3 before writing descriptors
- Needs token counts per call from Member 1 to complete the rewrite measurement

---

### QIAN RUOQI — Evaluation & Guardrail Lead

**Deliverables**

- D3(b) — ≥10 guardrail test cases:
  - Each must name the wrong behaviour it exists to catch and state the observed result
  - At least 3 must test hostile free-text in the claim narrative (prompt injection, instruction to approve, adversarial phrasing)
  - **All 10 run on scripted backend — free, no API key needed**
  - A scripted run proves the guardrail fires when the agent attempts the bad action; it cannot tell you whether a live model gets talked into it (that belongs to D5, not here)
- D4 — Evaluation set (40 cases, 8 negative):
  - Ordinary cases: 1 trial each
  - Negative cases: 3 trials each (2 extra per negative because these are the ones that flip between runs)
  - Total: 40 + (8 × 2) = 56 runs per model
  - Outcome-graded (ACT / ASK / ESCALATE) — grade what the agent concluded, not the path
  - Isolated — each case starts from a clean state
  - Mixed grading: code checks for decision and trigger fields; judgement checks for reason quality (name the judge model and keep grading prompt in repo)
  - Labels written in same shape as expected_outcomes_A.json (join on case_id)
  - At least one negative case that actually fired during development and changed something (earns explicit credit)

> **Every team member contributes evaluation cases.** 40 cases ÷ 6 members ≈ 7 cases each. Member 5 coordinates, owns the set, and ensures consistency.

**Live Model**
- Model D — different family from all above
- Runs v2 (final) prompt
- ~56 runs

**Report Sections**
- Section 3 — What the evidence showed (D4 + D5 pass rates, model divergence, what negative cases caught) · ~350 words — jointly with Member 6

**Dependencies**
- Needs scripted backend from Member 1 to validate cases before live battery runs
- Coordinates with Member 6 on how pass rate feeds layer 2 of the cost model

---

### FANG HIOIENG — Cost Analysis Lead

**Deliverables**

- D6 — Three-layer cost-to-serve model using **measured** numbers from D4 and D5 (not estimates):

  | Layer | What | Formula |
  |-------|------|---------|
  | 1 · Per-task variable | Input + output tokens at list price | input × price_in + output × price_out |
  | 2 · Expected fallback | Failure cost × failure rate | (1 − success rate) × US$7.60 |
  | 3 · Fixed monthly | Storage, infra, eval runs, monitoring | stated separately |

  Token formula: input ≈ B×T + D×T(T−1)/2

- Sensitivity table: cost per successful task at success rate ±10 percentage points; state whether conclusion survives the whole range
- Break-even success rate: p = 1 − (E − C) / F
  - C = one run cost on cheap model (tokens only)
  - E = layer 1 + layer 2 for expensive model
  - F = US$7.60 (claims assessor, 12 min @ US$38/hr)
  - Answer in one sentence: does the cheap model clear its break-even, or how far short does it fall?
- Four-lever cost ledger — before and after for each:

  | Lever | What it attacks | Where built | Report |
  |-------|----------------|-------------|--------|
  | 1 · Tool block size | B — re-sent every turn, linear | D2(a) | tokens of tool definitions before/after cuts |
  | 2 · Turn count | T — the quadratic term | D2(c) | turns and input tokens, sequential vs. parallel |
  | 3 · Observation size | D — compounds every later turn | D2(b) | tokens returned per call, v1 vs. v2 |
  | 4 · Success rate | Sets layer 2 — usually the biggest | D4 | measured pass rate and cost per successful task |

- State which lever dominated the bill and how you know
- State three caps that ship with the system: step cap, budget ceiling, monthly limit per user
- Note prompt caching only if measured (not assumed) — run with and without, take token counts from API response
- Report baseline first; adjusted figure (if caching used) beside it with explanation

**Live Model**
- Model E — cheap or mid tier, different family from all above
- Runs v2 (final) prompt
- ~56 runs

**Report Sections**
- Section 4 — What it costs (D6) · ~400 words
- Section 3 — jointly with Member 5

**Dependencies**
- Needs measured token counts and turn counts from Member 1
- Needs measured pass rate from Member 5 (D4)
- Cannot finalise numbers until both land — plan a handoff session before the final weekend

---

## What Every Member Must Do

| Task | Detail |
|------|--------|
| Write evaluation cases | ~7 cases each; Member 5 coordinates the full set of 40 |
| Run one live model | See Live Model column above — use your own OpenRouter key; same v2 prompt and same eval set across all members; only the MODEL string differs |
| Speak in the demo | 5-minute recorded demo; every member speaks; must show at least one negative case live |
| Complete peer rating | Due Wed 16 Sep 2026, 23:59 SGT — participation requirement |

---

## Model Assignment (coordinate before spending any live tokens)

| Member | Model | Tier | Family constraint |
|--------|-------|------|-------------------|
| SOOD VEDANSH | Model A | Cheap | — |
| YANG YICHEN | Model B | Mid | Different family from A |
| ZHANG QIZHI | Model C | Mid | Different family from A and B |
| QIAN RUOQI | Model D | Any | Different family from all above |
| FANG HIOIENG | Model E | Cheap or Mid | Different family from all above |
| CHEN BAIYI | v1 pass on Model A | Cheap | Same model as Member 1 — fixed |

> Models must span at least two price tiers. No two members may use models from the same family. Agree assignments at first team meeting before anyone spends live tokens.

---

## Dependency Order

```
ZHANG QIZHI
  └─ D0(c) committed to repo
       ↓
  SOOD VEDANSH
    └─ D1, D2(c), D3(a), D5(a), D7 (scripted backend)
         ↓ token counts, turn distribution, scripted run ready
         ├─ CHEN BAIYI ← D2(b) v1 run (cheap tier, same model as Member 1)
         ├─ YANG YICHEN ← D7 write-up (failure data)
         └─ QIAN RUOQI ← D3(b), D4 (scripted backend for guardrails)
                ↓ pass rate, eval results
                └─ FANG HIOIENG ← D6 (also needs token counts from Member 1)
```

---

## Key Dates

| Date | Milestone | Owner |
|------|-----------|-------|
| Before first agent commit | D0(c) "what good looks like" committed | ZHANG QIZHI |
| Wed 2 Sep, midday | Fixture data + scaffold on NTULearn | — |
| **Fri 4 Sep, 23:59 SGT** | TEAM_DECLARATION.docx submitted | All — FANG HIOIENG coordinates |
| By Fri 4 Sep | 40 evaluation cases written (even if not yet run) | QIAN RUOQI + all |
| Mon 7 / Tue 8 Sep | Class 6 — guardrails / OWASP (sharpens D3b) | — |
| **Sun 13 Sep, 23:59 SGT** | A2 due — repo, report, self-appraisal, video link | All |
| Wed 16 Sep, 23:59 SGT | Peer rating due | All |

---

## Hard Rules (from the Updated Brief)

- `BACKEND = "scripted"` must be the default in submitted code. Technical Execution is capped if it does not reproduce.
- D3(b), D5(a), and D7 all run on scripted backend — free. If you spend live tokens on any of these, stop.
- D0(c) must be committed before the first agent code commit. Commit timestamp is checked.
- v1 descriptor comparison runs on one model only (cheap tier, same as Member 1). Do not run v1 on all models.
- Tool names in Appendix A are suggestions. Agree final signatures with Members 1, 3, and 4 before descriptors are written.
- Debug on the scripted backend only. Live tokens are for the final battery.
- If estimated live spend exceeds US$3 per member, cut trials or move model down a tier — and say so in the report.
