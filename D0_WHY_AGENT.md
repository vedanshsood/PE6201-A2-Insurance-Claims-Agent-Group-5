# Section 1 · D0: Why an Agent at All
**Author**: ZHANG QIZHI (Member 3)  
**Track**: Problem A — Health-Insurance Claim First Response  
**Milestone**: Baseline Architectural Specification (Committed prior to Agent Implementation)  

---

## D0(a) · Ladder Placement & The Workflow Test

### 1. Defending Rung 7 (Agent) Across the Anthropic Ladder
We place the health-insurance claim triage engine strictly on **Rung 7 (Agent)** of the Class 4 ladder. Rungs 1 through 6 define deterministic or enumerated workflows where execution topology is fixed prior to runtime.

* **Why Rungs 1–6 Fail Problem A**:
  * **Rung 1 (Single Call) & Rung 2 (Prompt Chain)**: Incapable of resolving dynamic branching. An incoming claim carries an arbitrary number of line items (1 to 4+ procedures). A static chain cannot dynamically determine how many policy exclusions or pre-authorisations must be retrieved.
  * **Rung 3 (Routing) & Rung 4 (Parallelisation)**: Can only parallelize known, independent tasks or branch into mutually exclusive lanes. In Problem A, line-item checks exhibit runtime causal dependencies: whether to query pre-authorisation (`get_preauthorisation`) depends strictly on whether `check_coverage` flags that the procedure requires one.
  * **Rung 5 (Orchestrator-Workers) & Rung 6 (Evaluator-Optimiser)**: Multiply token overhead without resolving state transitions for asymmetric outcomes (e.g., a claim with two approved procedures and one excluded procedure processed in a single disposition).
* **The Cost of Rung 7**:
  * Step count (T) becomes a runtime variable, causing context prefix history to accumulate quadratically ($B \times T + D \times T(T-1)/2$).
  * Unbounded execution trajectories risk infinite loops and tool chatter, requiring code-level step caps and deterministic fallback gates.

---

### 2. The Four-Question Workflow Test

| Diagnostic Question | Workflow (Rungs 1–6) | Agent (Rung 7 — Our Build) | Observed Behavior in Problem A |
| :--- | :--- | :--- | :--- |
| **Who decides the sequence?** | Hardcoded by developer in advance | The model dynamically at runtime | Lapsed policy aborts at Turn 2; active policy proceeds to line-item adjudication. |
| **Does step count vary with input?** | No — fixed execution path | **Yes — fundamental discriminator** | 2 turns for policy breaches vs. 4–6 turns for multi-line claims requiring pre-auth verification. |
| **Can every path be tested?** | Yes — fully enumerable | No — unbounded permutations | Evaluated empirically on outcome distributions (N=30–50 evaluation set). |
| **Cost predictability** | Predictable (N fixed API calls) | Unpredictable until explicitly capped | Constrained via hard step caps ($T_{max} = 8$) and parallel tool dispatch. |

---

### 3. The Two Agentic Conditions
1. **Dynamic Trajectory**: The sequence of inspection steps cannot be known in advance because each claim presents distinct medical procedures, hospital networks, and attached documentation.
2. **Step-Level Ground Truth**: The model does not operate in an open-ended conversational vacuum; every step receives machine-speed structured telemetry from deterministic local records (e.g., policy expiry dates, exclusion tables) to correct hallucinated assumptions.

---

### 4. The Three-Question Test & The Governance Cliff

| Architecture Layer | Who picks what to retrieve? | Can it loop and re-query? | Can it change the world? |
| :--- | :--- | :--- | :--- |
| **Standard Retrieval (RAG)** | Hardcoded query string | No — single pass | No — read-only |
| **Agentic Retrieval** | Model at runtime | Yes — based on prior tool observations | No — read-only |
| **Full Agent (Our Build)** | Model at runtime | Yes — multi-turn loop | **YES — writes binding decision** |

* **The Governance Cliff**:
  * Read-only inspection tools (`get_claim`, `lookup_policy`, `check_coverage`, `get_preauthorisation`, `get_hospital_status`) operate purely within the Agentic Retrieval boundary and carry zero external liability.
  * The governance cliff is crossed at **`issue_decision_letter`**. Emitting a formal approval in principle creates legal and financial exposure for the insurer. Thus, the tool is strictly isolated behind a deterministic code gate (`autonomy="confirm"`) requiring explicit operator confirmation prior to persistence.

---

## D0(b) · When NOT to Build an Agent (Diagnostic Tests)

### Test 1 · The Ground-Truth Speed Test
An unsupervised loop is only viable if internal systems of record can contradict the model objectively at machine speed. Problem A connects to five deterministic local fixture tables:

* `members.json`: Resolves member identity and active policy IDs (<2ms).
* `policies.json`: Validates policy date ranges, deductible limits, and exclusion code lists (`EX-xx`) (<2ms).
* `procedures.json`: Supplies procedure descriptions and mandatory pre-authorisation flags (`requires_preauth`) (<2ms).
* `preauthorisations.json`: Matches pre-approval certificate validity against date of service (<2ms).
* `hospitals.json`: Validates panel accreditation status (<2ms).

Subjective human opinions are excluded from the loop runtime; all observations return structured JSON facts.

---

### Test 2 · The Reliability Arithmetic ($s = P^{1/T}$)

Across a multi-step trajectory, run-level pass rate $P$ compounds per-step reliability $s$ over $T$ turns:

$$P = s^T \iff s = P^{1/T}$$

* **Compounding Decay**:
  * If single-step reliability is $s = 0.960$ (96%), a compact 3-turn run achieves $P = 0.960^3 \approx 88.5\%$.
  * An undisciplined 12-turn sequential run degrades to $P = 0.960^{12} \approx 61.3\%$.
* **Operational Cost Link**:
  * In Problem A, each run failure escalates to a human claims assessor at a cost of $F = \$7.60$ (US$38/h $\times$ 12 min).
  * Total service cost is dominated by Layer 2 fallback:
    $$\text{Cost-to-Serve} = \text{Layer 1 (API tokens)} + (1 - P) \times \$7.60$$
  * Compressing turns via D2(c) parallel tool execution (collapsing sequential coverage checks into a single turn) directly reduces $T$, elevates $P$, and minimizes Layer 2 fallback penalties.
* **Diagnostic Boundary**:
  * $s$ is an empirical diagnostic, not a physical invariant.
  * Step quality varies: deterministic database lookups exhibit $s \approx 1.0$, whereas free-text narrative interpretation is prone to error.
  * Optimization requires raising $s$ via poka-yoke interface typing (Way 1) while simultaneously pruning $T$ via parallelization (Way 2).

---

## D0(c) · What Good Looks Like (5 Core Statements)
*These five testable criteria serve as the ground-truth benchmark for evaluation harness design (D4) and were committed prior to agent development.*

1. **Factual Grounding**: Every disposition is verifiable against fixture record fields (e.g., specific policy expiry dates, exclusion identifiers like `EX-14`), with zero ungrounded narrative fabrication.
2. **Protocol Adherence**: Concludes strictly in one of three standardized schema formats: `approve_in_principle` (with full itemized line breakdowns), `request_document` (explicitly naming the missing certificate and line item), or `escalate` (citing a single unambiguous trigger).
3. **Gated Integrity**: Calls `issue_decision_letter` at most once per trajectory, strictly after all necessary evidentiary dependencies are collected, and executes only when the autonomy gate criteria are fully satisfied.
4. **Adversarial Resilience**: Unconditionally escalates or refuses claims involving lapsed policies, exceeded annual limits, or prompt injection instructions embedded within user narratives, without executing unauthorized writes.
5. **Economic Viability**: Achieves an end-to-end cost per successful claim (Layer 1 + Layer 2) demonstrably lower than the human claims assessor benchmark of **US$7.60 per case**.

---

## D2(a) · Tool Set Selection & Justification Table

| Tool Name | Fails Without It? (Q1) | Confusable? (Q2) | Cost & Retention Justification (Q3) |
| :--- | :--- | :--- | :--- |
| `get_claim(claim_id)` | **Yes** — Sole entry point to resolve claim metadata & line items | **No** — Unique key & purpose | Mandatory prefix cost; executed on turn 1 |
| `lookup_policy(member_id)` | **Yes** — Sole source for policy validity & remaining limit | **No** — Partitioned by Member ID | Enables early exit on turn 2 if policy lapsed |
| `check_coverage(policy_id, code)` | **Yes** — Resolves exclusions & pre-auth requirement flags | **No** — Operates on line-level codes | Parallelizable across multiple line items in turn 2 |
| `get_preauthorisation(member_id, code)` | **Yes** — Validates active pre-approval certificates | **No** — Distinct conditional scope | Called only when `check_coverage` flags requirement |
| `get_hospital_status(hospital_id)` | **Yes** — Validates panel hospital network status | **No** — Independent entity schema | Parallelizable with policy and line coverage checks |
| `issue_decision_letter(...)` *(Gated)* | **Yes** — Sole write tool to record binding disposition | **No** — Only write tool in system | Irreversible write; covered by `autonomy="confirm"` |

### Pruned / Excluded Tools
* **Removed `search_policy_notes`**: Eliminates tool confusion against structured `check_coverage` lookups.
* **Removed `calculate_claim_totals`**: Financial summation is moved into deterministic Python code to prevent model arithmetic drift and unnecessary turn consumption.