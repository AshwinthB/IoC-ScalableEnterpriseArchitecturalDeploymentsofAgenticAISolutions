# AssetCareHQ — Enterprise Architecture Documentation

## 1. Executive Summary
**AssetCareHQ** is an Intelligent IT Asset Lifecycle & Operations Platform architected as an academic prototype for deterministic multi-agent orchestration. It demonstrates how autonomous agents can coordinate complex enterprise IT workflows—from asset procurement, inventory matching, and policy governance to risk evaluation, automated assignments, and lifecycle maintenance—while maintaining strict auditability and human-in-the-loop controls.

---

## 2. System Architecture Diagram

```mermaid
graph TB
    subgraph Client_Layer ["Client Layer (Presentation & UI)"]
        UI["Modern Enterprise React Web App (Vite + TS + Tailwind)"]
        Router["Client-Side Router & Session Guard"]
        Views["12 Operational Views (Dashboard, Inventory, Workflow, Approvals, Lifecycle, etc.)"]
        UI --> Router --> Views
    end

    subgraph Auth_RBAC ["Identity & Access Governance"]
        AuthContext["Auth Context & Token Guard"]
        Roles["RBAC Engine (Employee | Manager | Asset Admin)"]
        AuthContext --> Roles
    end

    subgraph Agent_Orchestration ["Deterministic Multi-Agent Engine"]
        Orchestrator["1. Orchestrator Agent (Workflow Coordinator & State Machine)"]
        PolicyAgent["2. Policy Agent (Compliance & Entitlement Engine)"]
        InventoryAgent["3. Inventory Agent (Multi-attribute Weighted Matching)"]
        RiskAgent["4. Risk Agent (LOW / MEDIUM / HIGH Triage)"]
        LifecycleAgent["5. Lifecycle Agent (Health, MTBF & Recurring Repair Tracker)"]
        AssignmentAgent["6. Assignment Agent (State Transition & Asset Reservation)"]

        Orchestrator --> PolicyAgent
        PolicyAgent --> InventoryAgent
        InventoryAgent --> RiskAgent
        RiskAgent --> LifecycleAgent
        LifecycleAgent --> AssignmentAgent
    end

    subgraph Simulated_Tools ["Service & Tool Abstraction Layer"]
        T1["Asset Search Tool"]
        T2["Asset Details Tool"]
        T3["Policy Check Tool"]
        T4["Risk Evaluation Tool"]
        T5["Asset Assignment Tool"]
        T6["Asset Transfer Tool"]
        T7["Asset Return Tool"]
        T8["Lifecycle Analysis Tool"]
        T9["Audit Logging Tool"]
        T10["Notification Tool"]
    end

    subgraph Data_Storage ["Data & State Persistence Layer"]
        DB[(AssetCareHQ Relational Store / Local Mock Engine)]
        T_Users["profiles & user_roles"]
        T_Assets["assets & asset_assignments"]
        T_Requests["asset_requests & approvals"]
        T_Transfers["transfers & repairs"]
        T_Audit["agent_events & audit_logs"]

        DB --- T_Users
        DB --- T_Assets
        DB --- T_Requests
        DB --- T_Transfers
        DB --- T_Audit
    end

    %% Interactions
    Views --> AuthContext
    Views --> Orchestrator
    PolicyAgent -.-> T3
    InventoryAgent -.-> T1
    RiskAgent -.-> T4
    LifecycleAgent -.-> T8
    AssignmentAgent -.-> T5
    Simulated_Tools --> DB
```

---

## 3. Multi-Agent Workflow Design

The core execution path for an asset request proceeds through a strict deterministic state machine:

```mermaid
stateDiagram-v2
    [*] --> RECEIVED: Request Submitted by Employee
    RECEIVED --> ANALYZING: Orchestrator Triage
    ANALYZING --> POLICY_CHECK: Policy Agent Evaluation
    
    state Policy_Check_Decision <<choice>>
    POLICY_CHECK --> Policy_Check_Decision
    Policy_Check_Decision --> INVENTORY_MATCHING: Policy Validated
    Policy_Check_Decision --> RISK_EVALUATION: Policy Restriction Detected

    INVENTORY_MATCHING --> RISK_EVALUATION: Inventory Agent Ranks Assets
    
    state Risk_Decision <<choice>>
    RISK_EVALUATION --> Risk_Decision
    Risk_Decision --> APPROVED: LOW Risk (< ₹50,000 & Compliant)
    Risk_Decision --> APPROVAL_REQUIRED: MEDIUM Risk (>= ₹50,000)
    Risk_Decision --> APPROVAL_REQUIRED: HIGH Risk (Restricted / Policy Violation)

    APPROVED --> ASSIGNING: Automated Policy Trigger
    
    APPROVAL_REQUIRED --> ASSIGNING: Manager / Admin Approval Granted
    APPROVAL_REQUIRED --> REJECTED: Reviewer Rejection

    ASSIGNING --> COMPLETED: Asset Tag Reserved, State -> IN_USE
    REJECTED --> [*]
    COMPLETED --> [*]
```

### Deterministic Asset Matching Formula
The Inventory Agent ranks available candidates using multi-factor normalized scoring:
$$\text{Score} = (\text{RAM} \times 0.30) + (\text{Storage} \times 0.20) + (\text{CPU} \times 0.25) + (\text{Age} \times 0.15) + (\text{Location} \times 0.10)$$

### Lifecycle Health & Replacement Formula
The Lifecycle Agent calculates replacement urgency (0–100):
$$\text{Score} = \text{Age Score (25)} + \text{Warranty Expiry (20)} + \text{Repair Frequency (20)} + \text{Health Degradation (20)} + \text{Performance (15)}$$
- **0–39:** Healthy
- **40–69:** Monitor
- **70–100:** Replacement Recommended (Flags recurring repairs $\ge 3$ for identical failure modes).

---

## 4. Security & Access Governance Model

1. **Role-Based Access Control (RBAC):**
   - **Employee:** Submit requests, view personal assigned inventory, monitor workflow trace for personal tickets.
   - **Manager:** Employee privileges + review and approve/reject medium-risk tickets and inter-department asset transfers.
   - **Asset Admin:** Full supervisory access, manual asset assignments/returns/offboarding, role management, high-risk reviews, and demo scenario triggering.

2. **Immutable Audit Logging:**
   - Every state transition, agent tool invocation, human approval, asset status change, and transfer is logged to `audit_logs` with actor details, timestamps, and target resource identifiers.
   - Normal users are prevented from altering or purging historical audit logs.

3. **Safe Credentials & Mock Isolation:**
   - Zero hardcoded cloud secrets or private keys in the frontend source bundle.
   - Deterministic simulations eliminate runtime LLM cost, hallucination risk, and external API rate limit vulnerabilities.

---

## 5. Monitoring & Operational Metrics

The platform incorporates real-time operational telemetry across three analytical axes:
- **Agent Health & Execution Counts:** Run counts, success rates, and mean stage latencies (75–235ms simulation bounds).
- **Pipeline Throughput:** Open requests, pending human approvals, auto-approval ratio, and rejection rates.
- **Hardware Fleet Utilization:** Active vs. available inventory counts, warranty expiration pipeline, and proactive replacement candidates.

---

## 6. Deployment Strategy

- **Build Output:** Static, tree-shaken SPA bundle produced via Vite (`npm run build`).
- **Target Hosting:** Edge CDN platforms (Vercel, Netlify, Cloudflare Pages, or AWS S3 + CloudFront).
- **Environment Parity:** The application runs out of the box in both offline mock mode (using seeded local persistence) and connected mode (via Supabase PostgreSQL with Row Level Security).
