# D6 — Cost-to-Serve Analysis

## Scope, method and assumptions

This analysis prices **Problem A: Health-Insurance Claim First Response** at the brief's volume of **8,000 claims/month**. It uses the five 60-trial V2 live batteries for success rates and input/output token counts. Each failed claim is escalated to a claims assessor at **US$38/hour for 12 minutes**, so `failure_cost = 38 × 12/60 = US$7.60`.

The baseline follows the Class 5 escalation model, not a retry model:

`Layer 1 = (measured input tokens × published list input price + measured output tokens × published list output price) / 60`

`Layer 2 = (1 − measured pass rate) × US$7.60`

`cost per successful task = Layer 1 + Layer 2`

`monthly cost = cost per successful task × 8,000 + Layer 3`

The list-price snapshot was retrieved from the [OpenRouter model catalogue](https://openrouter.ai/api/v1/models) on 20 September 2026. No cache-control field is sent in the supplied request payload. However, automatic provider caching cannot be excluded from a provider-reported `usage.cost`; therefore, that field is **not used** in the baseline. The reported-cost discrepancy is shown below and must be resolved by a controlled cache-on/cache-off experiment before any caching claim is made.

Layer 3 is **US$0 in this baseline only**: the submitted system uses local fixtures/scripted development and the supplied evidence records no paid storage, infrastructure, monitoring, maintenance, or recurring evaluation service. This is not a claim that production operations have zero fixed cost; any subsequently chosen paid service or staff-maintenance allowance must be added transparently to Layer 3.

## 1. Live-battery cost baseline and deployment choice

| V2 model | Pass rate (n=60) | Input / output tokens (60 trials) | List prices: input / output per token | Layer 1: list-price API/trial | Layer 2: expected escalation | Cost/successful task | Monthly cost at 8,000 claims |
|---|---:|---:|---:|---:|---:|---:|---:|
| **openai/gpt-oss-20b** | **54/60 = 90.0%** | 908,279 / 62,170 | US$0.00000003 / US$0.00000013 | **US$0.000589** | **US$0.760000** | **US$0.760589** | **US$6,084.71** |
| qwen/qwen-2.5-72b-instruct | 53/60 = 88.3% | 898,540 / 16,334 | US$0.00000036 / US$0.00000040 | US$0.005500 | US$0.886920 | US$0.892420 | US$7,139.36 |
| google/gemini-2.5-flash | 51/60 = 85.0% | 845,112 / 13,194 | US$0.00000030 / US$0.00000250 | US$0.004775 | US$1.140000 | US$1.144775 | US$9,158.20 |
| deepseek/deepseek-chat-v3.1 | 51/60 = 85.0% | 878,462 / 25,152 | US$0.00000025 / US$0.00000095 | US$0.004058 | US$1.140000 | US$1.144058 | US$9,152.47 |
| z-ai/glm-4.5-air | 49/60 = 81.7% | 858,694 / 91,902 | US$0.00000013 / US$0.00000085 | US$0.003162 | US$1.393080 | US$1.396242 | US$11,169.94 |

**Decision.** Deploy `openai/gpt-oss-20b` for the current system. It is both the cheapest list-price API option and the highest-scoring model in the battery. Its monthly baseline is US$6,084.71 plus any future Layer 3 charge. The result is dominated by error escalation: its plain-token API component is US$4.71/month at 8,000 claims, while expected escalations contribute US$6,080.00/month (99.92% of the variable total).

| Model | Plain list-price cost for 60 trials | Provider-reported `usage.cost` | Difference (reported − plain) |
|---|---:|---:|---:|
| openai/gpt-oss-20b | US$0.035330 | US$0.027361 | −US$0.007969 |
| qwen/qwen-2.5-72b-instruct | US$0.330008 | US$0.330008 | US$0.000000 |
| google/gemini-2.5-flash | US$0.286519 | US$0.191462 | −US$0.095057 |
| deepseek/deepseek-chat-v3.1 | US$0.243510 | US$0.149273 | −US$0.094237 |
| z-ai/glm-4.5-air | US$0.189747 | US$0.103527 | −US$0.086220 |

This validates why the baseline must use token counts × list price. The discrepancy may reflect provider discounts, caching, or another billing rule; the supplied data do not identify which, so it is not interpreted as a saving.

## 2. Cost-lever ledger

| Lever | Evidence / before → after | Cost implication |
|---|---|---|
| **1. Tool block size B** | The team rejected a seventh `lookup_required_documents` tool. The rejected definition would add **142 tokens every turn**; instead, `required_document` was added to the existing `check_coverage` return. | The avoided 142-token prefix is re-sent on every model turn. This is a documented design delta, not a separately provider-billed A/B run; it should not be presented as an observed dollar saving. |
| **2. Turn count T** | Scripted, paired 60-record audit at cap 32: **323 → 239 total backend turns** (mean **5.3833 → 3.9833**, −26.01%); serialised input characters **3,750,048 → 2,653,604** (−29.24%). Decision labels remained 57/60 in both conditions. | This meets the updated D2(c) requirement: state the dependency rule, measure both modes, and show that correctness did not change. Parallel calls remove 84 model round trips without changing tool work. The character counts establish direction only; they are not billed tokens or dollar savings and are not used in Layer 1. At cap 8, batching also removed one cap stop (serial 1; batched 0). |
| **3. Observation size D** | `check_coverage` V1 → V2 bare JSON mean: **24.46 → 31.97 tokens/call** across 67 observations (**+7.51; +30.7%**). V2 added `required_document`. | This is the bounded *returned-observation* measurement required for the descriptor rewrite; it should not be confused with end-to-end billed prompt tokens. End-to-end Layer 1 is independently priced from the D5 live-battery input/output counts above. `CLM-8901` improved from V1 failure to **3/3 V2 passes**. |
| **4. Success rate** | The deployed model scored **54/60 = 90.0%**. Its matched one-shot V1→V2 comparison was **33/42 → 39/42** (+14.3 pp); the full V2 battery is the cost baseline. | Success rate controls Layer 2. At 90%, expected escalation is **US$0.7600/task**, overwhelmingly larger than plain-token API cost (US$0.000589/task); it is therefore the dominant lever. |

The apparent tension between levers 1/3 and 4 is intentional: V2 adds a small, bounded observation field, but improves access to ground truth. In this system, reducing failure rate is worth much more than minimising API tokens. Under the 1 September document update, D2(c)'s central test is the dependency rule plus paired correctness evidence—not a prescribed live-model A/B run.

## 3. Sensitivity analysis — deployed `openai/gpt-oss-20b`

The uncertainty tested is success rate ±10 percentage points around the observed 90.0%; Layer 1 is held at the computed plain-list-price US$0.000589/trial and Layer 3 remains zero under the stated baseline assumption.

| Success rate | Layer 1 | Layer 2 | Cost/successful task | Monthly cost at 8,000 claims |
|---:|---:|---:|---:|---:|
| 80.0% | US$0.000589 | US$1.520000 | US$1.520589 | US$12,164.71 |
| **90.0% (measured)** | **US$0.000589** | **US$0.760000** | **US$0.760589** | **US$6,084.71** |
| 100.0% | US$0.000589 | US$0.000000 | US$0.000589 | US$4.71 |

The deployment recommendation survives the full range: `gpt-oss-20b` is still cheaper than every alternative at their measured performance. However, the absolute monthly bill is not robust: a 10-percentage-point fall in success rate almost doubles monthly cost. Monitoring and evaluation should therefore focus first on pass rate, especially negative cases.

## 4. Break-even success rate

For a conservative comparison, use the cheapest model (`gpt-oss-20b`) against the next-best all-in alternative, `qwen/qwen-2.5-72b-instruct`:

- Cheap-model token cost, `C` = US$0.000589.
- Qwen all-in cost, `E` = US$0.005500 + `(1 − 0.8833) × 7.60` = **US$0.892420**.
- Failure cost, `F` = **US$7.60**.
- Break-even success rate = `1 − (E − C)/F` = **88.27%**.

`gpt-oss-20b` measured **90.0%**, which is **1.73 percentage points above** its 88.27% break-even. Qwen is the highest-scoring alternative (88.3%) and its list-price API cost is 9.34× `gpt-oss-20b`'s, making it the strongest available comparator. The current battery contains no model that exceeds `gpt-oss-20b`'s 90.0% success rate, so no non-dominated “more accurate but more expensive” comparison can honestly be constructed from these data. This conclusion should be re-tested when data, prompts, or policy rules change.

## 5. Cost controls that ship with the system

| Control | Current evidence-backed setting | Cost role |
|---|---:|---|
| Step cap | **8 turns** | Stops runaway loops. The paired audit shows batched paths finish within five turns; serial execution can require nine. |
| Per-task budget ceiling | **60,000 tokens** | Guardrail blocks a run once cumulative usage exceeds this ceiling. |
| Per-user monthly limit | **Not implemented / not evidenced in the supplied artefacts** | **Outstanding D6 control.** A policy limit must be selected, implemented and guardrail-tested before this can be described as a shipped system control. |

## 6. Caching and reasoning-model treatment

No caching experiment or reasoning-token configuration/result was supplied. Accordingly, the baseline does **not** claim a caching discount or a reasoning-token adjustment. If either is introduced, it must be measured on the same evaluation set using API usage fields, with and without the feature, and reported beside—not substituted for—the plain-token baseline.

## Evidence sources

- Five supplied V2 live-battery notebooks: pass rates, prompt/completion tokens and harness-measured API costs.
- `D2(a) Tool Set Chosen, Not Collected.md`: avoided seventh-tool prefix addition of 142 tokens/turn.
- `D2b_EN - observation size D - tokens.md`: V1/V2 observation-token measurements and matched comparison.
- `D2c_Execution_Results_EN.md` and `运行结果汇总.json`: paired sequential/batched scheduling audit.
- OpenRouter model catalogue snapshot, retrieved 20 September 2026: plain input/output list prices used in Layer 1.
- `PE6201_A2_Document_Updates.pdf` (1 September 2026): D2(c) clarification that the required evidence is a dependency rule, measurement in both conditions, and unchanged correctness; D5 is the live-model battery.
