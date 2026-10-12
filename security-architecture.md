# Security Architecture

[**← Back to Portfolio Index**](README.md) · [**Microsoft Sentinel →**](sentinel.md) · [**AI Agents →**](ai-agents.md) · [**Books & Learning →**](books-and-learning.md) · [**Experience & Impact →**](experience-and-impact.md)

---

## Architectural Philosophy & Domain Scope

My security architecture work translates business risk, regulatory obligations, and infrastructure constraints into **Azure Cloud Adoption Framework (CAF) landing zones**, **Zero Trust identity and network boundaries**, **data encryption hierarchies**, and **governed security operations**.

Rather than treating security controls as isolated checklists, every design connects four layers:

```mermaid
flowchart LR
    A["1. Governance & Policy: Management Groups and Azure Policy"] --> B["2. Identity & Zero Trust: Conditional Access, PIM, FIDO2 MFA"]
    B --> C["3. Network & Data Perimeter: Hub-Spoke, Firewall, Private Link, CMK"]
    C --> D["4. SecOps & Assurance: Unified Defender, Sentinel, AMA/DCR, SOAR"]
```

---

## 1. Public Flagship Example: Enterprise AI/DR Architecture Comparison & RAG Security Boundary (`fde-lab`)

- **Visitor Access:** **Public source and demo instructions available now** in [`86sunbot/fde-lab`](https://github.com/86sunbot/fde-lab) ([`microsoft_vs_crowdstrike_ai_dr_guide.md`](https://github.com/86sunbot/fde-lab/blob/main/microsoft_vs_crowdstrike_ai_dr_guide.md), [`v3-enterprise-assistant/docs/security.md`](https://github.com/86sunbot/fde-lab/blob/main/document-assistant/versions/v3-enterprise-assistant/docs/security.md), and [`v3-enterprise-assistant/app/security.py`](https://github.com/86sunbot/fde-lab/blob/main/document-assistant/versions/v3-enterprise-assistant/app/security.py)).

### Problem and Intended Users
- **Problem:** Enterprise security architects and CISOs face two simultaneous challenges when adopting generative AI and modernizing SecOps:
  1. Selecting and integrating **AI-Driven Detection and Response (AI/DR)** control planes across identity, browser DLP, agent runtime execution, and SIEM data-lake economics (**Microsoft Defender XDR + Sentinel + Entra Agent ID + Purview** vs. **CrowdStrike Falcon + Next-Gen SIEM + Falcon Guardian**).
  2. Designing concrete API security boundaries (constant-time authentication, sliding-window rate limiting, input validation, and non-root container isolation) when evolving an internal Retrieval-Augmented Generation (RAG) prototype into a service (`v1 → v3`).
- **Intended Users:** CISOs, Cloud & AI Security Architects, SOC Transformation Leads, and Applied AI Engineers.

### Architecture Diagram
```mermaid
flowchart TD
    subgraph ControlPlanes ["Enterprise AI/DR Control-Plane Comparison (microsoft_vs_crowdstrike_ai_dr_guide.md)"]
        direction LR
        MS["Microsoft Security Stack<br/>• Identity: Entra Agent ID + Conditional Access<br/>• Data/Browser: Purview Endpoint DLP + DSPM for AI<br/>• Runtime: Prompt Shields + Defender BehaviorInfo<br/>• SecOps & FinOps: Sentinel + Security Copilot + APIM Quotas"]
        CS["CrowdStrike Falcon Stack<br/>• Identity: Falcon Identity Service-Account Monitoring<br/>• Data/Endpoint: Falcon Data Security<br/>• Runtime: Falcon AIDR + Kernel-Level Falcon Guardian<br/>• SecOps & Telemetry: Next-Gen SIEM + Charlotte AI + Token Counts"]
    end

    subgraph RAGService ["Executable RAG Service Security Boundary (v3-enterprise-assistant)"]
        Client["Streamlit UI / API Consumer"] -->|"X-API-Key Header"| Auth["verify_api_key (secrets.compare_digest)"]
        Auth -->|"Per-Key / Client-IP Bucket"| RL["SlidingWindowRateLimiter (HTTP 429 + Retry-After)"]
        RL -->|"Pydantic Schema & Length Bounds"| Engine["Hybrid TF-IDF + Vector Retrieval & Grounded Answer"]
    end

    ControlPlanes --> RAGService
```

### Representative Artifacts Visitors Can Inspect Now
1. **Enterprise AI/DR Comparative Architecture Guide (491 lines, 7 Mermaid diagrams):**
   - [**`fde-lab/microsoft_vs_crowdstrike_ai_dr_guide.md`**](https://github.com/86sunbot/fde-lab/blob/main/microsoft_vs_crowdstrike_ai_dr_guide.md) — Covers the *Three Jobs of AI Security* (AI for SecOps, Security for AI, Governance of AI), Shadow AI workstation/browser controls, AI agent identity governance (`Entra Agent ID` vs. service accounts), token cost control vs. security telemetry, **4 end-to-end attack scenarios** (*Sensitive Data Pasted into Public AI*, *Stolen API Key & Denial-of-Wallet*, *Indirect Prompt Injection via Email*, and *Compromised AI Agent Running Rogue Database Queries*), a 30/60/90-day rollout roadmap, and a 20-item assessment checklist.
2. **Executable API Security Boundary & ADRs (`v3-enterprise-assistant`):**
   - [**`document-assistant/versions/v3-enterprise-assistant/app/security.py`**](https://github.com/86sunbot/fde-lab/blob/main/document-assistant/versions/v3-enterprise-assistant/app/security.py) — Implements `SlidingWindowRateLimiter` (thread-safe `deque` per API key or client IP returning `429 Too Many Requests` with `Retry-After`) and `verify_api_key` using `secrets.compare_digest(provided_key, expected_key)`.
   - [**`document-assistant/versions/v3-enterprise-assistant/docs/security.md`**](https://github.com/86sunbot/fde-lab/blob/main/document-assistant/versions/v3-enterprise-assistant/docs/security.md) & [**`docs/decisions/ADR-001`..`ADR-006`**](https://github.com/86sunbot/fde-lab/tree/main/document-assistant/versions/v3-enterprise-assistant/docs/decisions) — Threat model and architectural decision records.

### Setup and Reproduction Instructions
```bash
git clone https://github.com/86sunbot/fde-lab.git
cd fde-lab/document-assistant/versions/v3-enterprise-assistant
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements-dev.txt
ruff check .
pytest -q
```

### Validation Evidence and Known Limitations
- **Validation Evidence:** **`32` automated `pytest` tests defined and passing (`32 passed`)** across 9 test files in `document-assistant/versions/v3-enterprise-assistant/tests/` (specifically including [`tests/test_security.py`](https://github.com/86sunbot/fde-lab/blob/main/document-assistant/versions/v3-enterprise-assistant/tests/test_security.py) for API-key rejection, constant-time comparison, and rate-limit exhaustion), plus `ruff check .` (`0` errors) and Docker build verification in [`.github/workflows/v3-ci.yml`](https://github.com/86sunbot/fde-lab/blob/main/document-assistant/versions/v3-enterprise-assistant/../../.github/workflows/v3-ci.yml).
- **Known Limitations:**
  - `microsoft_vs_crowdstrike_ai_dr_guide.md` is a comparative architectural guide and decision framework based on documented platform capabilities, not an empirical throughput or latency benchmark.
  - `v3-enterprise-assistant` uses an in-memory per-process rate limiter (`SlidingWindowRateLimiter`) and static environment-variable `X-API-Key` suitable for single-container prototypes; multi-instance deployments require a shared gateway (such as Azure API Management with Entra ID JWT validation and Redis-backed rate limiting).

### Contribution and Third-Party Attribution
- **My Contribution:** Authored the 491-line `microsoft_vs_crowdstrike_ai_dr_guide.md` architecture comparison, the `v1 → v3` Document Assistant progression, the FastAPI security boundary (`app/security.py`), the ADRs (`ADR-001`..`ADR-006`), and the 32-test `pytest` suite.
- **Third-Party Attribution:** The sample PDF used for offline retrieval chunking in `v2-ai-assistant/document.pdf` and `v3-enterprise-assistant/document.pdf` is a 4-page print-to-PDF excerpt of public Microsoft Learn documentation (*"Microsoft Defender for Identity Overview"*).

---

## 2. Azure Security Architecture Framework (`azure-security-framework`)

- **Visitor Access:** **Public case study with implementation currently private** (`86sunbot/azure-security-framework` is currently private and scheduled for Stage B public release).

### Problem and Audience
- **Problem:** Enterprise cloud teams frequently deploy Azure workloads before establishing consistent management-group policy inheritance, Conditional Access persona separation, private-link DNS resolution, or encryption key-management boundaries.
- **Audience:** Cloud Security Architects, Azure Platform Engineers, Security Assessors, and CISOs designing or reviewing Azure landing zones.

### Current Capabilities (Documented in Case Study)
- **Identity (`identity/conditional-access-policies.md`):** Persona-based Conditional Access design matrix covering administrators, standard users, guest/external identities, workload identities, break-glass exclusions, session controls, and phased `Report-only` → `On` rollout.
- **Network (`network/hub-spoke-architecture.md`):** Hub-and-spoke reference topology integrating Azure Firewall inspection, Bastion administrative access, Application Gateway WAF ingress, NSG micro-segmentation, and Private Endpoint / Private DNS integration.
- **Data Protection (`data-protection/encryption-at-rest.md`):** Comparative architecture and Bicep patterns for Platform-Managed Keys (PMK), Customer-Managed Keys (CMK) in Azure Key Vault / Managed HSM, Customer-Provided Keys (CPK), Azure Disk Encryption vs. Encryption at Host, and TDE for SQL workloads.
- **Compliance & Policy (`compliance/azure-policy-library.md`):** Management Group policy assignment strategy using `Audit`, `Deny`, `DeployIfNotExists` (DINE), and `Modify` effects aligned to Microsoft's MCSB control baseline.
- **DevSecOps (`devsecops/pipeline-security.md`):** Supply-chain and CI/CD pipeline security controls covering secret scanning, SAST, SCA, IaC policy gates (`Checkov`/`TFSec`), container image signing, and workload identity federation (OIDC).
- **Security Monitoring (`monitoring/sentinel-workspace-design.md`):** Single- vs. multi-workspace Microsoft Sentinel architecture, retention tiering, RBAC operating models, and Azure Lighthouse cross-tenant management.

### Maturity / Status
- **Reference Architecture Documentation (Active, v1.1 Remediated · Existing Root `MIT` License Preserved):** Structured Markdown reference guides with embedded Bicep, JSON policy definitions, KQL queries, and architecture diagrams.

### Evidence and Limitations
- **Validation Scope & Documented Pattern Scan:** 6 domain architecture documents mapped to Microsoft's MCSB control baseline, Azure CAF, and NIST SP 800-53 / CIS Controls. No automated runtime test suite is defined; embedded JSON/Bicep/KQL snippets and internal links were inspected statically, and regex pattern scans across `HEAD` (`b4dc780`) and Git history detected no authentication credentials or personal Azure GUIDs (only RFC 4122 zero/example GUIDs `00000000-...` / `11111111-...` and `contoso.com` domains).
- **Known Limitations:**
  - This repository provides reference patterns, decision matrices, and template snippets rather than an automated end-to-end Terraform/Bicep deployment module (for executable Sentinel deployment automation, see [`Sentinel-as-a-Service`](sentinel.md#3-sentinel-as-a-service--greenfield-deployment-kit--orchestrator-sentinel-as-a-service)).
  - Additional planned deep-dive modules (such as standalone PIM and Defender for Cloud CSPM guides) are consolidated in the companion [*Microsoft Security Architect Master Blueprint v1.1*](books-and-learning.md#1-microsoft-security-architect-master-study-guide--reference-architecture-v11).

### Canonical Source & Access Status
- **Access Status:** **Public case study with implementation currently private** (`86sunbot/azure-security-framework`, prepared on local branch `portfolio/remediation-2026-10` at commit `b4dc780` — Stage B candidate).
- **Related Books & Templates:** Cross-referenced with [*Microsoft Security Architect Master Blueprint v1.1*](books-and-learning.md#1-microsoft-security-architect-master-study-guide--reference-architecture-v11) and [*SC-500 Complete Study Guide v1.1*](books-and-learning.md#2-microsoft-sc-500-cloud-and-ai-security-engineer--complete-study-guide-v11).

---

## 3. Enterprise Security Architecture Templates & Lab Runbooks (`ms-security-architect-study-guide`)

- **Visitor Access:** **Public case study and chapter/template summary with manuscript currently private** (unpublished local v1.1 manuscript; Stage C companion candidate).

### Problem and Audience
- **Problem:** Security architects often lack standardized, review-board-ready templates for High-Level Designs (HLDs), Architecture Decision Records (ADRs), and repeatable tenant lab validation from identity through AI security.
- **Audience:** Practicing Security Architects, Cloud Solution Architects (CSA-Security), and technical leads preparing Architecture Review Board (ARB) packages.

### Current Capabilities (Documented in Case Study)
- **20-Section Enterprise HLD Template (`templates/Enterprise-HLD-Template.md`):** Covers executive summary, business drivers, scope/out-of-scope boundaries, assumptions/constraints, current-state baseline, target architecture topology, identity/network/data/compute/SecOps control planes, failure-mode analysis, FinOps/TCO modeling, phased migration plan, and operational handover criteria.
- **Architecture Decision Record Standard (`templates/Architecture-Decision-Record-ADR-Template.md`):** Structured trade-off evaluation framework capturing context, options considered, security/latency/cost trade-offs, decision rationale, and compensating controls.
- **Contoso-ZeroTrust Integrated Lab Runbook (`labs/Contoso-ZeroTrust-Lab-Runbook.md`):** Six dependency-ordered enterprise validation labs spanning CAF landing zones, Entra ID Conditional Access + FIDO2 + PIM, Hub-Spoke + Azure Firewall Premium + WAF, Purview DLP + Defender CNAPP, Unified SecOps (`security.microsoft.com`) + AMA/DCR + FinOps TCO modeling, and Private Azure OpenAI RAG + Entra Agent ID.

### Maturity / Status
- **Reference Architecture & Lab Runbook Pack (Frozen v1.1 Local Manuscript Baseline, October 2026):** Complete documentation, templates, and lab scripts accompanying the 958-page *Microsoft Security Architect Master Blueprint*.

### Evidence and Limitations
- **Verified Evidence:** 109 documented ADRs, 15 reusable architecture templates, 150 rendered Mermaid diagrams, and 6 end-to-end lab runbooks validated against October 2026 Microsoft Learn documentation.
- **Known Limitations:** Lab runbooks are designed for execution in a dedicated non-production Azure/Entra test tenant and require appropriate Azure subscription quotas and Entra ID P2 / Defender licensing for full feature execution.

### Canonical Source & Access Status
- **Access Status:** **Public case study with manuscript currently private**; see [**`books-and-learning.md#1-microsoft-security-architect-master-study-guide--reference-architecture-v11`**](books-and-learning.md#1-microsoft-security-architect-master-study-guide--reference-architecture-v11) for complete chapter and appendix specifications.

---

## Cross-Links to Related Categories

- **SIEM Architecture, Log Onboarding Assurance & Greenfield Deployment:** See [**`sentinel.md`**](sentinel.md)
- **AI Security Threat Models (`ai-security-lab`) & Governed Agent Architectures:** See [**`ai-agents.md`**](ai-agents.md)
- **CISSP Domain Architecture Mappings & Study Books:** See [**`books-and-learning.md`**](books-and-learning.md)
- **Career Roles & Credentials:** See [**`experience-and-impact.md`**](experience-and-impact.md)
