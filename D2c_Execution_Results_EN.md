# D2(c) Execution Results

## 1. Experimental Setup and Counting Method

| Item | Configuration |
|---|---|
| Test subject | Project v2 code snapshot and data supplied with the attachments |
| Execution mode | Deterministic `ScriptedBackend`; no live LLM API calls |
| Data provenance | Supplementary offline experiments conducted by Codex during report preparation |
| Original execution time | 2026-09-17 10:30:58 UTC |
| Latest verification | Rerun in a temporary copy on 2026-09-19; all four summary tables and individual execution traces matched the saved results, and all script assertions passed |
| Cases and repetition rule | 42 unique cases: 33 cases with an expected approval decision were run once each, and 9 cases with other expected decision labels were run three times each, producing 60 execution records |
| Sequential mode | Execute only the first call in each list returned by the backend, then proceed to the next turn |
| Batched mode | Execute the complete list of calls returned by the backend in each turn |
| Other settings | Identical prompt, tools, data and scripted backend; `confirmed=True` |

**Turn count T is the number of calls to `backend.next_move()`, including the turn that returns the final decision.** Executing three read-tool calls within a single turn counts as one turn and three read-tool calls. The gated write performed by the application after the final decision does not add a backend turn. Batching here reduces backend round trips; the tool functions themselves are still executed one by one in a Python loop.

## 2. Main Results: Complete Execution Paths (Turn Cap of 32 in Both Conditions)

| Metric | Sequential | Batched |
|---|---:|---:|
| Execution records | 60 | 60 |
| Total backend turns | 323 | 239 |
| Mean backend turns | 5.3833 | 3.9833 |
| Median backend turns | 5 | 4 |
| Maximum turns observed in the test set | 9 | 5 |
| Total read-tool calls | 263 | 263 |
| Executions stopped by the turn cap | 0 | 0 |
| Decision-label matches | 57/60 | 57/60 |
| Decision-label match rate | 95.00% | 95.00% |
| Cumulative serialized input characters | 3,750,048 | 2,653,604 |
| Live-model input/output tokens | Not measured | Not measured |
| Live-model execution cost and latency | Not measured | Not measured |

Calculated results that can be cited:

- Total turns decreased by **84 turns**: `(323 − 239) / 323 = 26.01%`.
- The mean reduction was **1.40 turns per execution record**: `84 / 60 = 1.40`.
- Cumulative serialized input characters decreased by **29.24%**. This is a character-count reduction, not a measured reduction in tokens or cost.
- Across all 60 paired records, case/trial identifiers, tool execution records and final output objects matched exactly. Tool execution records include tool names, arguments, return values and execution order.

Both conditions produced decision-label mismatches in the three repeated records for `CLM-8952`. Therefore, 95% is only the decision-label match rate for these offline experiments. It does not establish correctness across all business rules, amounts or supporting evidence, and it is not a measured accuracy score for a live LLM.

## 3. Results Under the Default Turn Cap of 8: Evidence for the Step-Cap Discussion

| Metric | Sequential | Batched |
|---|---:|---:|
| Total backend turns | 322 | 239 |
| Mean backend turns | 5.3667 | 3.9833 |
| Median backend turns | 5 | 4 |
| Maximum observed turns | 8 (affected by truncation) | 5 |
| Total read-tool calls | 263 | 263 |
| Executions stopped by the turn cap | 1 | 0 |
| Decision-label matches | 56/60 | 57/60 |
| Decision-label match rate | 93.33% | 95.00% |
| Cumulative serialized input characters | 3,733,540 | 2,653,604 |

In sequential mode, `CLM-Z002` requires a ninth turn to produce the final decision and therefore stops when the cap is 8. This difference shows that batching can reduce turn-cap stops on this test set; it should not be interpreted as an improvement in the script's business reasoning.

**Do not mix the two tables when writing the report.** The 26.01% reduction in total turns comes from the main comparison with a cap of 32. The results under the default cap of 8 include one truncated sequential execution.

## 4. Worked Example for D2(c): CLM-8842

| Action | Sequential turn(s) | Batched turn |
|---|---:|---:|
| `get_claim` | 1 | 1 |
| `lookup_policy` | 2 | 2 |
| `get_hospital_status` | 3 | 2 |
| `check_coverage` × 3 | 4, 5, 6 | 3 |
| `get_preauthorisation` | 7 | 4 |
| Return `final` | 8 | 5 |

Both conditions execute seven read-tool calls for this case, while the turn count decreases from **8 to 5**. The case completes under both turn-cap settings.

Dependency explanation:

1. `get_claim` provides the member, hospital and claim-line information required for subsequent lookups.
2. `lookup_policy` and `get_hospital_status` do not depend on each other's results and can therefore be placed in the same turn.
3. This implementation obtains `policy_id` from the policy lookup result, so coverage queries must occur in a later turn. Coverage queries for different procedures can be issued as a batch.
4. `get_preauthorisation` waits for the coverage results to identify which procedures require preauthorisation.
5. After the backend returns the final decision, the application performs the gated write.
