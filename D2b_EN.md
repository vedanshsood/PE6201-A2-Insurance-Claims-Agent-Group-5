# D2(b): descriptors, poka-yoke, v1 vs v2 rewrite

## 1. Six-field tool descriptor contracts

| Name + signature | WHAT | INPUT | RETURNS (bounded) | FAILS WHEN | Irreversible? |
|---|---|---|---|---|---|
| `get_claim(claim_id: ClaimId) -> ClaimObservation \| null` | Retrieves the unique claim and the evidence that starts all routing paths. | Exact claim identifier; an unknown or malformed ID returns `null`. | One claim object: member, hospital, service date, lines, documents, narrative and `duplicate_of`; fixture maximum: 4 lines and 2 documents. | No matching claim exists. | No |
| `lookup_policy(member_id: MemberId) -> PolicyObservation \| null` | Retrieves the member's linked policy, including validity, limits and exclusions. | Exact member ID from `get_claim`; unknown ID returns `null`. | One `{member, policy}` object; no list or free-text search. | Member or linked policy is absent. | No |
| `check_coverage(code: ProcedureCode, policy_id: PolicyId) -> CoverageResult \| null` | Returns authoritative requirements for one procedure under one policy. | Exact claim-line code and policy ID from `lookup_policy`; unknown input returns `null`. | Exactly one 5-field object: procedure code, coverage status, preauthorisation flag, required document/null, exclusion rule/null. | Policy or procedure is absent. | No |
| `get_preauthorisation(member_id: MemberId, procedure_code: ProcedureCode, date_of_service: ISODate) -> list[Preauthorisation] \| null` | Determines whether an authorisation is valid on the service date. | Exact values from earlier evidence. | Matching records only; at most 2 in the fixtures, or `null` when absent. | No match or invalid ID; an expired record is evidence, not an error. | No |
| `get_hospital_status(hospital_id: HospitalId) -> HospitalObservation \| null` | Returns the authoritative panel status and country. | Exact hospital ID from `get_claim`. | One object: hospital ID, name, panel and country. | No matching hospital exists. | No |
| `issue_decision_letter(claim_id, decision, lines_resolved, approved_total, refused_total=0) -> DecisionRecord` | Records the simulated final decision. The model cannot call it. | Application-supplied, validated canonical final decision only. | One confirmation record: claim, decision, line resolutions, totals and gate setting. | Confirmation absent, duplicate action, or deterministic guardrail failure. | **Yes** - `confirm` gate, validation and de-duplication |

## 2. Poka-yoke changes

| Design change | Error made impossible |
|---|---|
| Exact typed IDs replace names as tool arguments. | Selecting the wrong member, policy, hospital or procedure because a name is ambiguous or mistyped. |
| `required_document: str \| null` is returned explicitly in V2 coverage evidence. | The interface failing to present the procedure-specific document requirement needed to identify a missing-document case. |
| Gated write removed from the model-visible tool set; application code controls it. | Direct or duplicate irreversible decision recording by the model. |

## 3. Observation-size data: `check_coverage`

**Measurement unit:** compact serialized JSON characters (`json.dumps(..., separators=(',', ':'))`), calculated over all 67 claim-line coverage observations in the 42-case fixture set. These are payload-size measurements, **not model-token counts**.

| Tool version | Return shape | Calls measured | Mean chars/call | Median chars/call | Min-max chars/call | Change from V1 |
|---|---|---:|---:|---:|---:|---:|
| V1 | `code`, `covered`, `requires_preauthorisation`, `exclusion_rule` | 67 | 88.7 | 87 | 86-112 | - |
| V2 | `procedure_code`, `coverage_status`, `requires_preauthorisation`, `required_document`, `exclusion_rule` | 67 | 140.1 | 135 | 134-160 | +51.4 (+58.0%) |

### Observation-size interpretation

| Fact | Value |
|---|---:|
| Added V2 field | `required_document` |
| Added serialized payload per coverage call, mean | 51.4 characters |
| Maximum V2 coverage observation | 160 characters |
| Fixture claims | 42 |
| Claim-line coverage observations in fixture set | 67 |
| Maximum claim lines in one claim | 4 |

### Other bounded observation sizes in the supplied fixtures

| Tool | Return bound | Min / mean / max serialized characters |
|---|---|---:|
| `get_claim` | One claim; at most 4 lines and 2 documents | 193 / 252.9 / 370 |
| `lookup_policy` | One `{member, policy}` object | 283 / 300.0 / 391 |
| `get_preauthorisation` | At most 2 matching records | max 253 |
| `get_hospital_status` | One hospital object | max 81 |

## 4. Matched one-shot V1-to-V2 comparison

| Metric | V1 - Chen Baiyi, 42 cases | V2 - Sood Vedansh, first trial of the same 42 cases | Change |
|---|---:|---:|---:|
| Passing executions | 33/42 | 39/42 | +6 |
| Pass rate | 78.6% | 92.9% | +14.3 pp |
| Failed executions | 9 | 3 | -66.7% |
| Total live tokens | 316,526 | 692,307 | +375,781 |
| Mean tokens/execution | 7,536 | 16,483 | +8,947 |
| Total API cost | US$0.009017 | US$0.019865 | +US$0.010848 |
| Mean cost/execution | US$0.000215 | US$0.000473 | +US$0.000258 |
| Mean turns/execution | 4.79 | 4.60 | -0.19 |
| Mean latency/execution | 22.27 s | 42.15 s | +19.89 s |
| Scripted guardrail checklist | 10/10 | 10/10 | unchanged |

## 5. Full V2 model battery (context only)

| Model | Pass rate | Negative pass rate | Total cost | Mean latency |
|---|---:|---:|---:|---:|
| `openai/gpt-oss-20b` | 54/60 (90.0%) | 21/27 (77.8%) | US$0.027361 | 42.91 s |
| `qwen/qwen-2.5-72b-instruct` | 53/60 (88.3%) | 20/27 (74.1%) | US$0.330008 | 13.94 s |
| `google/gemini-2.5-flash` | 51/60 (85.0%) | 18/27 (66.7%) | US$0.191462 | 3.50 s |
| `deepseek/deepseek-chat-v3.1` | 51/60 (85.0%) | 20/27 (74.1%) | US$0.149273 | 23.27 s |
| `z-ai/glm-4.5-air` | 49/60 (81.7%) | 19/27 (70.4%) | US$0.103527 | 16.94 s |

## 6. Comparison constraints

| Constraint | Handling |
|---|---|
| V1 has 42 one-shot executions; V2 has 60 trials because negative cases are repeated. | The matched table uses the first V2 trial for each of the same 42 case IDs. Do not compare 33/42 with 54/60 directly. |
| V2 alters more than descriptor wording. | Attribute results to the V2 tool-interface and prompt package: return shape, descriptors, system prompt and empty-output handling all changed. |
| Per-tool observation tokens were not logged by the live notebooks. | Use the observation-size table only as serialized-character payload data. Do not relabel it as tokens. |
