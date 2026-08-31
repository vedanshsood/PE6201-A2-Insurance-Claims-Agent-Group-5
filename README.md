# PE6201 A2 — Applied AI System
## Team [C-5] · Problem A: Health-Insurance Claim First Response

---

## Task Distribution

| # | Member | Deliverables Owned | Report Section(s) | Dependencies |
|---|--------|--------------------|--------------------|--------------|
| 1 | **SOOD VEDANSH** (Lead Coder) | D1 — ReAct loop implementation<br>D2(c) — multi-tool-call parsing, dependency rule, sequential vs. parallel measurement<br>D3(a) — code-layer guardrails (step cap, budget ceiling, de-duplication, autonomy gate)<br>D5 — scripted backend + live OpenRouter model battery<br>D7 — technical reproduction of both failures | — | None (produces core outputs others depend on) |
| 2 | **YANG YICHEN** (Support Coder) | Code cross-checks and review<br>Supports D0(a) — ladder/rung selection and architecture choice<br>Supports D2(c) — dependency rule design<br>Supports D3(a)/D7 — stop-condition and step-cap sizing | Section 5 (draft, with Member 1)<br>Section 6 (draft) | Works alongside Member 1 |
| 3 | **ZHANG QIZHI** | D0 — why an agent, ladder placement, ground-truth signals, "what good looks like" (5 statements)<br>D2(a) — tool set selection + 3-question justification table | **Section 1** (Why an agent + tool set) | None — should be completed before agent code begins |
| 4 | **CHEN BAIYI** | D2(b) — six-field descriptor contracts per tool, poka-yoke identification, descriptor rewrite (v1 vs. v2) | **Section 2** (Tool layer) | Needs token counts / pass rates from Member 1 for the rewrite measurement |
| 5 | **QIAN RUOQI** | D3(b) — 10+ guardrail test cases (named failure mode + observed result)<br>D4 — 30–50 evaluation cases, including 6–10 negative cases | **Section 3** (What the evidence showed) — jointly with Member 6 | Needs scripted/live run results from Member 1 |
| 6 | **FANG HIOIENG** | D6 — three-layer cost model, sensitivity table, break-even success rate, four-lever cost ledger | **Section 4** (What it costs) | Needs measured token counts, turn counts (Member 1) and pass rate (Member 5/D4) |

---

## Outstanding Items (to confirm ownership)

- [ ] **Report Section 5** (The two failures) — draft: Member 2, technical detail: Member 1
- [ ] **Report Section 6** (What we would not deploy) — draft: Member 2
- [ ] **Team Declaration** (`TEAM_DECLARATION.md`) — due Fri 4 Sep 2026, 23:59 SGT
- [ ] **Team Self-Appraisal** — completed together, checked into submission folder
- [ ] **Recorded Demonstration** (5 min, every member speaks, negative case shown live) — coordinate: [Name]
- [ ] **Final report assembly** (2,000-word cap, tables/figures excluded) — coordinate: [Name]

---

## Key Dates

| Date | Milestone |
|------|-----------|
| Fri 4 Sep 2026, 23:59 SGT | Team declaration due |
| Mon 7 / Tue 8 Sep 2026 | Class 6 (guardrails, OWASP) |
| Sun 13 Sep 2026, 23:59 SGT | **A2 due** |
| Wed 16 Sep 2026, 23:59 SGT | Peer rating due |

---

## Notes

- All members must be able to explain any block of code submitted, regardless of who wrote it.
- Commit history must corroborate this contribution table — commit early and often under your own name.
- D0's "what good looks like" statements (Member 3) must be committed **before** the first agent code commit.
- Cost analysis (Member 6) and evaluation write-up (Member 5) are dependent on Member 1's harness output — plan working sessions accordingly rather than working in isolation until the deadline.
