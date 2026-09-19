# Section 2: The Tool Layer

## 2.1 D2(a) Tool Set Chosen, Not Collected

Our six-tool suite (`get_claim`, `lookup_policy`, `check_coverage`, `get_preauthorisation`, `get_hospital_status`, and the gated `issue_decision_letter`) was strictly audited against Class 4’s three design criteria:

| Tool | Fails Without It? | Confusable? | Why It Earns Its Place |
| :--- | :--- | :--- | :--- |
| `get_claim` | **Yes** — Sole entry point for claim lines and metadata. | **No** | Anchors execution to ground truth. |
| `lookup_policy` | **Yes** — Validates coverage dates and annual limits. | **No** | Enables early turn-2 escalation on lapsed policies. |
| `check_coverage` | **Yes** — Resolves line-level coverage, pre-auth, and document requirements. | **No** | Prevents confident hallucinations on procedure eligibility. |
| `get_preauthorisation`| **Yes** — Verifies procedure pre-approval validity dates. | **No** | Conditionally queried only when pre-auth is mandatory. |
| `get_hospital_status` | **Yes** — Confirms hospital network panel status. | **No** | Parallelizable with policy checks in turn 2. |
| `issue_decision_letter`| **Yes** — Sole write action recording the binding adjudication. | **No** | **Gated action**; hidden from model schemas, executed by code under `confirm`. |

### Design Restraint & Pruning (The "Try Not Adding a Tool" Heuristic)
Applying the "try not adding a tool" heuristic, we rejected creating a 7th tool (`lookup_required_documents`) to resolve V1’s failure on `CLM-8901`. Adding another tool definition would have expanded prompt prefix overhead ($B$, Cost Lever 1) by 142 tokens every turn and added sequential turns. Instead, we expanded `check_coverage` to surface `required_document` directly. This restraint bounded prefix costs, maintained tool discriminability, and converted `CLM-8901` to a 100% pass rate in V2 across all 3 trials.
