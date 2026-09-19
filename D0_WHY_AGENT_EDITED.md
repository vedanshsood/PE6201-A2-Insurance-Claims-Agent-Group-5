# Section 1 · D0: Why an Agent at All
**Author**: ZHANG QIZHI (Member 3)  
**Track**: Problem A — Health-Insurance Claim First Response  
**Milestone**: Baseline Architectural Specification  

---

## D0(a) · Ladder Placement & The Workflow Test

### 1. Defending Rung 7 (Agent) Across the Anthropic Ladder
We place the health-insurance claim response system strictly on **Rung 7 (Agent)** of the Class 4 autonomy ladder. Rungs 1 through 6 represent static, enumerable workflows where execution sequence and step count are fixed in advance:

* **Why Rungs 1–6 Fail Problem A**:
  * **Rungs 1–2 (Single Call & Prompt Chains)**: Cannot handle dynamic branching. A claim contains arbitrary line items (1 to 4+ procedures), each requiring distinct exclusion and document evaluations that fixed prompt sequences cannot resolve.
  * **Rungs 3–4 (Routing & Parallelisation)**: Only parallelize isolated checks or route to static lanes. In Problem A, procedure verification exhibits runtime conditional dependencies: calling `get_preauthorisation` strictly depends on whether `check_coverage` returns `requires_preauthorisation = true`.
  * **Rungs 5–6 (Orchestrator-Workers & Evaluator-Optimiser)**: Multiply token overhead without resolving state transitions for asymmetric dispositions (e.g., partly payable claims where two lines are covered and one is excluded within a single decision letter).
* **The Cost of Rung 7**:
  * Turn count ($T$) becomes a runtime variable, causing prefix token history to compound quadratically ($B \times T + D \times T(T-1)/2$).
  * Unconstrained loops introduce risks of infinite invocation and unauthorized write actions, requiring hard code-level step caps and human-in-the-loop gates.

---

### 2. The Four-Question Workflow Test

| Diagnostic Question | Workflow (Rungs 1–6) | Agent (Rung 7 — Our Build) | Observed Behavior in Problem A |
| :--- | :--- | :--- | :--- |
| **Who decides the sequence?** | Hardcoded in advance | Model dynamically at runtime | Aborts at Turn 2 on lapsed policy; proceeds to procedure checks on active policy. |
| **Does step count vary with input?** | No — fixed path | **Yes — fundamental discriminator** | 2 turns for policy breaches vs. 4–6 turns for multi-line claims requiring pre-authorisation. |
| **Can every path be tested?** | Yes — fully enumerable | No — unbounded permutations | Evaluated empirically on outcome distributions across our 60-trial evaluation set. |
| **Cost predictability** | Predictable ($N$ fixed calls) | Unpredictable until capped | Bounded via hard step cap (`MAX_TURNS=8`) and execution budget ceiling. |

---

### 3. The Two Agentic Conditions
1. **Dynamic Trajectory**: Subsequent steps cannot be predetermined; execution paths are decided dynamically based on claim line items and preliminary lookups.
2. **Step-Level Ground Truth**: The model does not generate arbitrary narratives; every step is grounded against deterministic local fixtures to correct assumptions at machine speed.

---

### 4. The Three-Question Test & The Governance Cliff

| Architecture Layer | Who picks what to retrieve? | Can it loop and re-query? | Can it change the world? |
| :--- | :--- | :--- | :--- |
| **Standard Retrieval (RAG)** | Hardcoded query | No — single pass | No — read-only |
| **Agentic Retrieval** | Model at runtime | Yes — based on prior tool output | No — read-only |
| **Full Agent (Our Build)** | Model at runtime | Yes — multi-turn loop | **YES — records binding adjudication** |

* **The Governance Cliff**:
  * Read-only inspection tools (`get_claim`, `lookup_policy`, `check_coverage`, `get_preauthorisation`, `get_hospital_status`) operate within the safe Agentic Retrieval boundary with zero mutation risk.
  * The governance cliff occurs at **`issue_decision_letter`**, where an irreversible financial commitment is recorded. To prevent unauthorized execution, this action is excluded from model-visible schemas and isolated behind an application code gate (`AUTONOMY="confirm"`).

---

## D0(b) · When NOT to Build an Agent (Diagnostic Tests)

### Test 1 · The Ground-Truth Speed Test
An unsupervised loop is justifiable only when authoritative systems of record contradict the model at machine speed. Problem A connects to local structured tables:
* `members.json` & `policies.json`: Validate membership, coverage dates, and annual limits (<2ms).
* `procedures.json` & `required_documents.json`: Supply pre-auth flags and mandatory documents (<2ms).
* `preauthorisations.json`: Matches pre-approval validity dates against the date of service (<2ms).
* `hospitals.json`: Validates panel accreditation status (<2ms).

Subjective narrative assertions are falsified instantly against structured table facts.

---

### Test 2 · The Reliability Arithmetic ($s = P^{1/T}$)

Across a multi-step trajectory, run-level pass rate $P$ compounds per-step reliability $s$ over $T$ turns ($P = s^T \iff s = P^{1/T}$).

* **Empirical Measurement**:
  In our V2 evaluation using `qwen/qwen-2.5-72b-instruct` across 60 trials, the agent achieved an end-to-end pass rate of $P = 88.33\%$ (53 passing runs) over an average horizon of $T = 5.47$ tool calls (328 total calls across the battery). This derives an empirical single-step reliability of:

$$s = (0.8833)^{1 / 5.47} \approx 0.9776 \quad \text{(97.76\%)}$$

* **Operational Cost Link**:
  Each failed claim escalates to a human assessor at $F = \$7.60$ (US$38/h $\times$ 12 min). Because service cost is dominated by Layer 2 fallback ($\text{Cost-to-Serve} = \text{Layer 1} + (1 - P) \times \$7.60$), compressing turns via parallelization directly elevates $P$ and curbs costly human fallbacks.
* **Diagnostic Boundary**:
  $s$ is a diagnostic tool, not a physical constant. Deterministic lookups exhibit $s \approx 1.0$, whereas free-text interpretation carries lower fidelity. System optimization requires raising step reliability via strict schema validation while pruning trajectory length ($T$).

---

## D0(c) · What Good Looks Like (5 Core Statements)

1. **Factual Grounding**: Every disposition is verifiable against fixture record fields, with zero invented policy or coverage facts.
2. **Protocol Adherence**: Concludes strictly in one of three standardized outcomes: `approve_in_principle`, `request_document` (naming the specific missing item and affected line), or `escalate` (citing a single trigger).
3. **Gated Integrity**: The gated write (`issue_decision_letter`) executes at most once per trajectory, strictly after all evidentiary facts are established, and requires confirmation.
4. **Adversarial Resilience**: Deterministically escalates claims containing hostile narrative injections, policy date breaches, or duplicate claim IDs without following adversarial prompts.
5. **Economic Viability**: Operates with a cost per successful claim (Layer 1 + Layer 2) demonstrably below the human claims officer benchmark of **US$7.60 per case**.
