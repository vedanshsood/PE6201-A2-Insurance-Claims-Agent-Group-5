# D2(c) Dependency Rule - Problem A

Owner support: YANG YICHEN.

## Rule

A pair of tools may run in the same turn only when neither tool needs the other's output. If one tool supplies an id, date, flag, or routing fact required by another tool, the second tool must wait for the next turn.

## Problem A Tool Dependencies

| Tool | May run with | Must wait for | Reason |
|---|---|---|---|
| `get_claim(claim_id)` | Nothing | Initial request only | It returns `member_id`, `hospital_id`, `date_of_service`, documents, narrative, and `lines`. Every later lookup depends on this record. |
| `lookup_policy(member_id)` | `lookup_hospital`, all independent `check_coverage` calls, and possibly `check_duplicate_claim` | `get_claim` | It needs `member_id` from the claim. It does not need hospital or coverage results. |
| `lookup_hospital(hospital_id)` | `lookup_policy`, all independent `check_coverage` calls, and possibly `check_duplicate_claim` | `get_claim` | It needs `hospital_id` from the claim. It does not decide coverage itself. |
| `check_coverage(code, policy_id)` | Other `check_coverage` calls for other lines, `lookup_policy`, `lookup_hospital`, and possibly `check_duplicate_claim` | `get_claim`; policy id from `lookup_policy` or known member-policy join | Each line can be checked independently once the policy and line codes are known. |
| `get_preauthorisation(member_id, procedure_code, date_of_service)` | Other preauthorisation calls for other lines that are already known to require preauth | `get_claim` and `check_coverage` | It should be called only for line items where `check_coverage` says `requires_preauth=True`. Calling it for every line wastes turns and can blur the decision logic. |
| `check_duplicate_claim(member_id, hospital_id, date_of_service, lines)` | `lookup_policy`, `lookup_hospital`, and `check_coverage` after `get_claim` | `get_claim` | It needs all four matching facts: member, hospital, date of service, and lines. It does not need coverage results. |
| `issue_decision_letter(...)` | Nothing | All required checks and the autonomy gate | This is the gated write. It must happen at most once and only after the evidence is complete. |

## Defensible Parallel Grouping

For a typical multi-line claim such as `CLM-8842`:

| Turn | Calls | Why this grouping is valid |
|---|---|---|
| 1 | `get_claim(claim_id)` | Must run alone because every later call needs fields from the claim. |
| 2 | `lookup_policy(member_id)` + `lookup_hospital(hospital_id)` + `check_duplicate_claim(...)` + `check_coverage(...)` once per line | These all depend on the claim, but not on each other. The coverage checks are line-independent. |
| 3 | `get_preauthorisation(...)` only for lines requiring it | This depends on the coverage result, so it cannot be in turn 2. |
| 4 | `issue_decision_letter(...)` | Gated action. It uses the final decision and evidence trail. |

## Sequential-vs-Parallel Measurement Plan

Run the same evaluation set twice:

| Mode | Expected difference | What must not change |
|---|---|---|
| Sequential | More turns, more repeated prompt/history input, higher token cost. | Expected decisions, triggers, and required record fields. |
| Parallel | Fewer turns by grouping independent calls. | Pass rate should stay the same; any change must be explained case by case. |

Report these fields from the harness:

| Metric | Sequential | Parallel | Notes |
|---|---:|---:|---|
| Trials | TODO | TODO | Same evaluation set and same prompt version. |
| Pass rate | TODO | TODO | Correctness should not move. |
| Median turns | TODO | TODO | Used in D0(b) and D7. |
| Worst-case turns | TODO | TODO | Used to defend the step cap. |
| Input tokens | TODO | TODO | Main D2(c) cost evidence. |
| Output tokens | TODO | TODO | Should be recorded, though input usually dominates repeated history. |
| Cost per run | TODO | TODO | Use measured API usage for live runs; scripted counts are only reproducibility evidence. |

## Claim for Report Section 2

Our dependency rule keeps `get_claim` and `issue_decision_letter` as single turns, but collapses independent post-claim lookups into one turn. The main saved cost is not that fewer tool calls are made; the same work is done. The saving comes from sending the accumulated trajectory back to the model fewer times. The one boundary we deliberately do not cross is preauthorisation: it waits until `check_coverage` identifies which lines require it.
