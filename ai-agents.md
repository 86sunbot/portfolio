# AI Agents & Applied Engineering

[**← Back to Portfolio Index**](README.md) · [**Security Architecture →**](security-architecture.md) · [**Microsoft Sentinel →**](sentinel.md) · [**Books & Learning →**](books-and-learning.md) · [**Experience & Impact →**](experience-and-impact.md)

---

## Engineering Philosophy: Bounded AI & Deterministic Controls

When applying Large Language Models (LLMs) and AI agents to security operations or regulated workflows, optimistic prompt engineering is not an architecture. My applied AI engineering centers on **explicit architectural boundaries**:

1. **Deterministic Verification Gates:** Model outputs (such as KQL detection rules or structured clinical observations) are treated as untrusted candidate artifacts that must pass schema validation, syntax/AST verification, and invariant checks before execution.
2. **Explicit Epistemic States:** Systems distinguish between verified positive state (`satisfied` / `PASS`), verified negative state (`not-satisfied` / `FAIL`), and missing or degraded upstream telemetry (`unknown` / `DEGRADED`)—failing closed rather than guessing.
3. **Human-in-the-Loop (HITL) Governance:** High-impact actions (such as promoting a Microsoft Sentinel analytics rule or releasing a patient batch) require human approval backed by tamper-evident validation and review evidence.

```mermaid
flowchart LR
    A["1. Untrusted Input: Workspace Schema, Reports, Legacy State"] --> B["2. Bounded AI Layer: Candidate Generation and Extraction"]
    B --> C["3. Deterministic Gate: KQL Schema Check, Invariants, Backtest"]
    C --> D["4. Human Review & Execution: SHA-256 Gate, Disabled Deploy, Audit Chain"]
```

---

## Public Flagship Example: Governed AI Sentinel Rule-Generation Workflow (`fde-lab/sentinel-usecase-deployment`)

- **Visitor Access:** **Public source and demo instructions available now** in [`86sunbot/fde-lab/sentinel-usecase-deployment`](https://github.com/86sunbot/fde-lab/tree/main/sentinel-usecase-deployment) ([`workflow.py`](https://github.com/86sunbot/fde-lab/blob/main/sentinel-usecase-deployment/src/sentinel_usecase/workflow.py), [`openai_proposer.py`](https://github.com/86sunbot/fde-lab/blob/main/sentinel-usecase-deployment/src/sentinel_usecase/agent/openai_proposer.py), [`gateway.py`](https://github.com/86sunbot/fde-lab/blob/main/sentinel-usecase-deployment/src/sentinel_usecase/deployment/gateway.py), and [`Smoke-Test-SentinelDeployment.ps1`](https://github.com/86sunbot/fde-lab/blob/main/sentinel-usecase-deployment/scripts/Smoke-Test-SentinelDeployment.ps1)).

### Problem and Intended Users
- **Problem:** Giving an LLM direct tool access to create or enable SIEM detection rules in Microsoft Sentinel risks deploying invalid KQL, bypassing peer review, or triggering alert storms. Security operations teams need an agent workflow where the LLM can only propose structured candidate artifacts, deterministic code enforces validation and single-pass repair, human operators must explicitly approve (`ApprovalDecision.APPROVE`), and deployment gateways re-validate artifacts and enforce `enabled = False` before invoking Azure deployment scripts.
- **Intended Users:** AI Security Engineers, SecOps Architects, and Detection Engineering Teams building human-governed AI workflows.

### Architecture Diagram
```mermaid
flowchart TD
    Req["1. Analyst Natural-Language Request (UseCaseWorkflow.generate)"] --> Route{"Pattern Supported? (Entra ID Password Spray)"}
    Route -->|"No"| Unsupported["Fail Closed: UnsupportedUseCaseError (No LLM Call or Deployment)"]
    Route -->|"Yes"| Propose["2. Untrusted Proposal Layer (OpenAIRuleProposer or ReferencePasswordSprayProposer - no tool execution)"]
    Propose --> Validate1["3. Deterministic Two-Layer Validation (RuleValidator: structural.py + detection.py)"]
    Validate1 -->|"is_valid == False"| Repair["Single-Pass Repair Attempt (proposer.propose with validation feedback)"]
    Repair --> Validate2["Re-Validate Candidate"]
    Validate1 -->|"is_valid == True"| Pending["STATUS_PENDING_HUMAN_APPROVAL (Review Artifact Written to generated-rules/)"]
    Validate2 -->|"is_valid == True"| Pending
    Validate2 -->|"is_valid == False"| Failed["STATUS_REJECTED_VALIDATION (Blocked from Approval/Deploy)"]
    Pending --> Human["4. Human Review Gate (ApprovalRecord / ApprovalDecision.APPROVE)"]
    Human -->|"UseCaseWorkflow.deploy"| Gateway["5. PowerShellCsvDeploymentGateway (gateway.py) - Re-verifies approval & validation, forces Enabled=false, runs Preview before -Apply"]
```

### Representative Artifacts Visitors Can Inspect Now
1. **Workflow Orchestrator, Single-Pass Repair & Pre-Deployment Gate (`workflow.py`):**
   - [**`fde-lab/sentinel-usecase-deployment/src/sentinel_usecase/workflow.py`**](https://github.com/86sunbot/fde-lab/blob/main/sentinel-usecase-deployment/src/sentinel_usecase/workflow.py) — Implements `UseCaseWorkflow.generate()` (unsupported request fail-closed check + single-pass LLM repair) and `UseCaseWorkflow.deploy()` (lines 53–65 verbatim below):
   ```python
   if approval.decision != ApprovalDecision.APPROVE:
       raise DeploymentBlockedError(
           f"Deployment blocked: human approval decision is '{approval.decision.value}', not 'approve'."
       )

   validation = self.validator.validate(rule)
   if not validation.is_valid:
       messages = "; ".join(f"[{issue.code}] {issue.message}" for issue in validation.errors)
       raise DeploymentBlockedError(f"Deployment blocked: rule failed validation: {messages}")

   if rule.enabled is not False:
       raise DeploymentBlockedError("Deployment blocked: rule must remain disabled (enabled=False).")
   ```
2. **PowerShell CSV Deployment Gateway & Smoke-Test Script (`gateway.py` & `scripts/`):**
   - [**`fde-lab/sentinel-usecase-deployment/src/sentinel_usecase/deployment/gateway.py`**](https://github.com/86sunbot/fde-lab/blob/main/sentinel-usecase-deployment/src/sentinel_usecase/deployment/gateway.py) — `PowerShellCsvDeploymentGateway` re-checks `approval.decision == ApprovalDecision.APPROVE` and `rule.enabled is False`, writes `generated-rules/deployment-input/<rule-id>/rule.kql` and `sentinel-use-cases.csv` (`"Enabled": "false"`), executes the external `Deploy-SentinelUseCases.ps1` script in preview mode first, and only runs `-Apply -Confirm:$false` if preview succeeds (see [`scripts/Smoke-Test-SentinelDeployment.ps1`](https://github.com/86sunbot/fde-lab/blob/main/sentinel-usecase-deployment/scripts/Smoke-Test-SentinelDeployment.ps1) and [`scripts/README.md`](https://github.com/86sunbot/fde-lab/blob/main/sentinel-usecase-deployment/scripts/README.md)).
3. **Additional Publicly Inspectable Regulated-Workflow Prototypes:**
   - [**`86sunbot/FDE-Final-Capstone`**](https://github.com/86sunbot/FDE-Final-Capstone) — CGT patient-to-batch orchestration console (`118` `pytest` tests, fail-closed `satisfied`/`not-satisfied`/`unknown` gates, bitemporal history, SHA-256 hash-chained SQLite audit log).
   - [**`86sunbot/caldera-monitoring-triage`**](https://github.com/86sunbot/caldera-monitoring-triage) — Clinical monitoring report triage prototype (`9` `pytest` tests on `main`; `10` Python + `3` TypeScript checks on [Draft PR #1](https://github.com/86sunbot/caldera-monitoring-triage/pull/1)).

### Setup and Reproduction Instructions
```bash
git clone https://github.com/86sunbot/fde-lab.git
cd fde-lab/sentinel-usecase-deployment
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt -r requirements-dev.txt
ruff check .
PYTHONPATH=src pytest -q tests
# Run end-to-end offline rule creation (writes review artifact to generated-rules/)
python run.py create "Create a password spray detection for Entra ID sign-in logs" --offline-reference
```

### Validation Evidence and Known Limitations
- **Validation Evidence:** **`28` automated `pytest` tests defined and passing (`28 passed in 0.04s`)** in [`sentinel-usecase-deployment/tests/`](https://github.com/86sunbot/fde-lab/tree/main/sentinel-usecase-deployment/tests), including [`test_workflow.py`](https://github.com/86sunbot/fde-lab/blob/main/sentinel-usecase-deployment/tests/test_workflow.py) (testing unsupported request fail-closed behavior, single-pass LLM repair, and approval state transitions) and [`test_deployment.py`](https://github.com/86sunbot/fde-lab/blob/main/sentinel-usecase-deployment/tests/test_deployment.py) (testing rejection of unapproved, rejected, tampered, or `enabled=True` rules, and verifying that live apply requires a prior preview pass). Wired into CI on [Draft PR #4](https://github.com/86sunbot/fde-lab/pull/4).
- **Known Limitations:**
  - The public `fde-lab/sentinel-usecase-deployment` package is the frozen **V1 baseline** supporting only the Entra ID single-IP password spray pattern and external PowerShell deployment (`Deploy-SentinelUseCases.ps1`).
  - Live OpenAI proposal generation (`OpenAIRuleProposer`) requires `OPENAI_API_KEY`, and live Sentinel rule creation requires `pwsh`, the external `Deploy-SentinelUseCases.ps1` script (`SENTINEL_DEPLOY_SCRIPT_PATH`), the `Az` PowerShell modules, and an authenticated Azure subscription; offline verification uses `--offline-reference` and mocked subprocess runners.

### Contribution and Third-Party Attribution
- **My Contribution:** Designed and implemented `UseCaseWorkflow`, `OpenAIRuleProposer`, `ReferencePasswordSprayProposer`, `PowerShellCsvDeploymentGateway`, `Smoke-Test-SentinelDeployment.ps1`, and the 28-test `pytest` suite.
- **Third-Party Attribution:** Uses standard open-source libraries (`openai`, `pydantic`, `pytest`, `ruff`); contains no third-party templates or proprietary code.

---

## 1. AI-Assisted Microsoft Sentinel Detection Engineering Agent (`ai-agent-usecase-deployment` V3.4)

- **Visitor Access:** **Public case study with V3.4 implementation currently private** (`86sunbot/ai-agent-usecase-deployment` V3.4 is currently private and scheduled for Stage C clean-companion release; the V1 baseline above is **publicly inspectable now** in [`86sunbot/fde-lab/tree/main/sentinel-usecase-deployment`](https://github.com/86sunbot/fde-lab/tree/main/sentinel-usecase-deployment)).

### Problem and Audience
- **Problem:** Extending the V1 single-pattern prototype to arbitrary enterprise detection requirements requires live Log Analytics workspace schema discovery, multi-table KQL generation, historical backtesting, and cryptographic readiness verification before deployment.
- **Audience:** Detection Engineers, SecOps Architects, and AI Security Engineering Teams.

### Current Capabilities (Documented in Case Study)
- **Workspace Telemetry Discovery & Grounding (`discovery.py`, `kql_sources.py`):** Queries live Log Analytics workspace schemas (or synthetic workspace snapshots) before rule generation so candidate KQL only references tables and columns verified to exist in the target environment.
- **Dual AI Provider + Offline Template Architecture (`generator.py`, `llm.py`):** Supports both live LLM generation (**OpenAI** and **Microsoft Foundry / Azure OpenAI**) and deterministic offline templates, normalizing outputs into a strict Pydantic `UseCaseArtifact` schema.
- **Deterministic KQL & Schema Validation (`validation.py`):** Inspects candidate rules for required MITRE ATT&CK tactic/technique mappings, entity mappings (`Account`, `IP`, `Host`, etc.), query window/frequency alignment, banned KQL performance anti-patterns (`search *`, unbounded joins), and schema column existence.
- **Historical Backtesting & Synthetic Telemetry (`backtest.py`, `synthetic.py`):** Evaluates candidate KQL against historical workspace telemetry or generated attack/benign synthetic events to measure expected alert volume and false-positive risk before deployment.
- **Cryptographic Readiness Gate & Safe Deployment (`readiness.py`, `deployment.py`, `terraform.py`):** Enforces a client deployment readiness gate that verifies SHA-256 digests of validation and human-review evidence artifacts before allowing ARM REST deployment or Terraform bundle export—and **always deploys rules with `enabled = false`** so human SOC operators control production activation.

### Maturity / Status
- **Working Engineering Prototype (V3.4 Specification-Driven Prototype, Locally Tested · License Selection Pending Owner Confirmation):** Python CLI (`python run.py ...`) verified locally across offline simulation, synthetic backtesting, and Microsoft Sentinel lab workflows (`test-artifacts/acceptance/2026-10-10-v3.4-lab-workflow.md`).

### Evidence and Limitations
- **Defined vs. Locally Executed Automated Tests:** **`296` automated tests defined** across 29 test files (`186` in `tests/` + `110` in `test-artifacts/`); **`295` locally executed and passed via `pytest` (`3.32s`)**, with **`1` live Azure Sentinel E2E test skipped** (`tests/test_e2e_live_sentinel.py`, gated behind `--run-live`).
- **Schema Validation vs. Live KQL Execution & Effectiveness:** Offline `validation.py` checks schema fields, table/column presence against a workspace snapshot, and static KQL anti-patterns (`search *`, missing `TimeGenerated`), while `backtest.py` replays rules against historical or synthetic fixtures. Neither offline check guarantees live Log Analytics query execution speed or production true-positive/false-positive effectiveness without staged shadow-mode tuning (`enabled = false`).
- **Documented Pattern Scan & Identifier vs. Credential Distinction:**
  - *Scan Scope:* Regex pattern scans across `HEAD` (`c8bb1c7`) and Git history detected **no authentication credentials/secrets** (no API keys, client secrets, or passwords).
  - *Non-Secret Cloud/Personal Identifiers:* 22 tracked files in `HEAD` (`generated-rules/2026-10-10-live-campaign/...`, `test-artifacts/...`, `scripts/...`, `CHANGELOG.md`) and Git history contain **non-secret personal lab identifiers** (Azure Subscription ID, Tenant ID, subscription display name, and personal Gmail address). Because `tests/test_readiness.py` verifies SHA-256 digests across those live campaign artifacts, Stage C publication will export a **clean-history public companion repository** with synthetic placeholder GUIDs (`00000000-...`) and regenerated synthetic provenance/hashes, keeping the private source repository intact.

### Canonical Source & Access Status
- **Access Status:** **Public case study with V3.4 implementation currently private** (`86sunbot/ai-agent-usecase-deployment`, prepared on local branch `portfolio/remediation-2026-10` at commit `c8bb1c7`; scheduled for clean-companion public release in Stage C).

---

## 2. Cell & Gene Therapy (CGT) Patient-to-Batch Orchestration (`FDE-Final-Capstone`)

- **Visitor Access:** **Public source and demo instructions** ([`86sunbot/FDE-Final-Capstone`](https://github.com/86sunbot/FDE-Final-Capstone)).

### Problem and Audience
- **Problem:** In autologous Cell & Gene Therapy workflows, an inherited legacy orchestration system (`source_baseline/`) suffered from silent fail-open exceptions, unverified chain-of-identity transitions, single-timestamp overwrites that erased late-arriving data history, and unauthenticated role elevation via caller-supplied headers.
- **Audience:** Forward Deployed Engineers, Systems Architects, and Engineering Hiring Managers evaluating brownfield-to-greenfield architecture and test discipline.

### Current Capabilities (Reconciled with Public `main` Code)
- **Fail-Closed State Machine & Three-State Verification (`model.py`, `application.py`):** Evaluates workflow transitions across explicit `satisfied`, `not-satisfied`, and `unknown` states—halting progression (`fail-closed`) whenever required clinical, manufacturing, cold-chain, or QA release gates are missing or stale.
- **Bitemporal State Reconciliation (`storage.py`, `patient_service.py`):** Tracks both *effective time* (when a clinical/lab event occurred) and *recorded time* (when the system ingested it), preserving historical state reconstruction when out-of-order updates arrive.
- **Tamper-Evident SHA-256 Hash-Chained Audit Log (`storage.py`):** Links every state transition and exception decision in a SQLite hash chain (`previous_hash` → `record_hash`) with automated chain-integrity verification (`verify_audit_chain()`).
- **Opaque Bearer-Token Mutation Auth & 6-Role / Scope RBAC (`api.py`, `security.py`):**
  - *Mutating API Authentication (`api.py`):* The slot-reservation endpoint (`POST /slots/reservations`) rejects caller-supplied role headers and authenticates `Authorization: Bearer <token>` via constant-time `secrets.compare_digest` against opaque server-side tokens mapped to `Principal(subject, roles, scopes)` in `FDE_DEMO_IDENTITIES_JSON` (failing closed with `HTTP 503` when unconfigured or `HTTP 401` on invalid token).
  - *Service-Layer Authorization (`security.py`):* Enforces **6 backend RBAC roles** (`ADMIN`, `COORDINATOR`, `IDENTITY_AUTHORITY`, `PLANNER`, `QUALITY_AUTHORITY`, `VIEWER`) and resource scopes (`*` or specific site/case scopes).
  - *Browser UI Lenses (`web/`):* Read-only status/source endpoints and the isolated 3-POC demo (`POST /api/demo/run`) require no mutation token and expose **7 read-only role/persona demo lenses** (Patient Operations, Identity, Logistics/Planning, Manufacturing, Lab/QC, Quality, and Executive views).

### Maturity / Status
- **Public Academic / Engineering Simulation Capstone (Locally Tested · Existing `source_baseline/LICENSE-SYNTHETIC-TRAINING.txt` Preserved; Root Application License Pending Owner Selection):** Built against a synthetic brownfield training dataset (`source_baseline/`). **Explicitly non-clinical, non-production, and not GxP-validated.**

### Evidence and Limitations
- **Defined vs. Executed Automated Tests:** **`118` automated tests defined** across 28 test files on public `main` (`eed0620`); **`118` executed and passed via `pytest`** with `96%` source coverage across `src/fde_capstone/` and `0` `ruff` errors.
- **Known Limitations (Documented in Repo):**
  - Uses deterministic rule/state evaluation (`AI_MODE=off` by default); no live LLM is in the critical safety path.
  - Of the 57 evaluation cases executed, 55 are internal structural passes, 0 failed, and 2 human-study cases are inconclusive (`7` original cases have full scoped property assertions, `9` are partial, and `39` extension cases are structural probes; `29/31` requirements internally verified, `0` production verified).
  - The SQLite audit chain is **tamper-evident** (detects unauthorized SQL mutation via SHA-256 hash verification), not hardware-immutable.
  - Contains third-party/coursework reference PDFs (`reference/FDE_PDF.pdf`, `archive/assessment-reports/*.pdf`) whose redistribution rights must be confirmed independently of any application code license.

### Canonical Source & Access Status
- **Access Status:** **Public source and demo instructions** ([**`https://github.com/86sunbot/FDE-Final-Capstone`**](https://github.com/86sunbot/FDE-Final-Capstone)).
- **Key Documentation:** [`FINAL_CAPSTONE_REPORT.md`](https://github.com/86sunbot/FDE-Final-Capstone/blob/main/docs/FINAL_CAPSTONE_REPORT.md) · [`CLIENT_DELIVERABLE_ACCEPTANCE_MATRIX.md`](https://github.com/86sunbot/FDE-Final-Capstone/blob/main/docs/CLIENT_DELIVERABLE_ACCEPTANCE_MATRIX.md) · [`Stage 2 Prototype Reuse Disposition`](https://github.com/86sunbot/FDE-Final-Capstone/blob/main/docs/stages/stage_02/05_ANTIGRAVITY_REUSE_DISPOSITION.md)

---

## 3. Clinical Monitoring Report Triage Prototype (`caldera-monitoring-triage`)

- **Visitor Access:** **Public source and demo instructions** ([`86sunbot/caldera-monitoring-triage`](https://github.com/86sunbot/caldera-monitoring-triage) · [Draft PR #1](https://github.com/86sunbot/caldera-monitoring-triage/pull/1) · [Sprint Walkthrough](https://github.com/86sunbot/caldera-monitoring-triage/blob/main/docs/FDE_SPRINT_STORY_AND_WALKTHROUGH.md) · [10-Minute Demo Script](https://github.com/86sunbot/caldera-monitoring-triage/blob/main/docs/DEMO_PRESENTATION_SCRIPT.md)).

### Problem and Audience
- **Problem:** Clinical trial study managers reviewing multi-page site monitoring reports face cognitive overload when manually triaging potential protocol deviations, overdue follow-up items, and site risk patterns—especially when upstream eTMF or site-registry services experience partial outages.
- **Audience:** AI Application Engineers, Regulated-Domain Product Teams, and FDE Reviewers.

### Current Capabilities (Public `main` Branch & Review PR #1)
- **Pre-GxP Cognitive Surfacing Architecture ([`ADR-001`](https://github.com/86sunbot/caldera-monitoring-triage/blob/main/docs/governance/decisions/ADR-001-surfacing-vs-predictive.md)):** Deliberately scoped as a non-binding, human-reviewed surfacing worksheet outside the GxP system-of-record boundary (`CandidateObservation.check_forbidden_fields` rejects `score`, `risk_level`, `priority`, `severity`, `rank`, and `deviation_confirmed`).
- **Verbatim Source Attribution (`SourceCitation` in `src/models.py` & `src/pipeline/extractor.py`):** Every surfaced observation records `document_id`, `page_number`, and `verbatim_sentence` extracted from the report page text (`report.pages`), grouped by site and theme into a `SameThemeAcrossVisitsView`.
- **Three Operational Resilience Controls ("Curveballs"):**
  - *Curveball 1 (EDC & Regional Boundary):* Rejects subject-level EDC clinical fields (`enforce_edc_boundary`) and drops restricted-region reports (`SITE-404`, `RESTRICTED_REGION_X`) before extraction ([`ADR-002`](https://github.com/86sunbot/caldera-monitoring-triage/blob/main/docs/governance/decisions/ADR-002-region-restriction-boundary.md)).
  - *Curveball 2 (Registry Fail-Closed Hold):* Holds documents in staging (`ExclusionRecord`) if the Site Registry service returns HTTP 500 rather than ingesting unverified regions.
  - *Curveball 3 (Denominator Disclosure):* Degrades batch status (`DEGRADED`) and displays exact retrieved/expected report counts (`11 of 20 reports`, `3 of 6 reports`) during partial eTMF HTTP 500 outages.

### Maturity / Status
- **Public FDE Sprint Prototype (Dual Python + TypeScript/React Implementation · License Selection Pending):** Interactive web prototype (`npm run dev` via `tsx server.ts` and `src/App.tsx`) and Python pipeline (`src/pipeline/triage_graph.py`) operating over 20 synthetic monitoring reports (`data/sample_reports/twenty_reports.json`).

### Evidence and Limitations
- **Public `main` (`eeac9b9`) vs. Review Draft PR #1 (`7c7d6e7`):**
  - **Public `main` Automated Tests (`9` defined / `9` passed via `pytest tests/ -v --tb=short`):** Covers regional restriction gating, EDC boundary enforcement, forbidden-score rejection, verbatim sentence/page provenance, Google AI Studio unconfigured fallback, 20-report end-to-end slice execution, and HTTP 500 eTMF / Site Registry degradation.
  - **Review Draft PR #1 ([`pull/1`](https://github.com/86sunbot/caldera-monitoring-triage/pull/1)):** Adds negative citation-provenance regression tests (`10` `pytest` tests total), automated TypeScript Curveball 1–3 verification (`scripts/verify_ts_controls.ts`), `npm run lint`, and `npm run build` in GitHub Actions, and removes the obsolete `src/main.py` reference from the README.
- **Known Limitations:**
  - **Not a GxP-validated system:** Per [`ADR-001`](https://github.com/86sunbot/caldera-monitoring-triage/blob/main/docs/governance/decisions/ADR-001-surfacing-vs-predictive.md), the prototype sits *outside* 21 CFR Part 11 / GxP validation as an unrecorded cognitive aid.
  - **In-memory execution:** Outputs are returned in-memory rather than persisted to WORM storage.

### Canonical Source & Access Status
- **Access Status:** **Public source and demo instructions** ([**`https://github.com/86sunbot/caldera-monitoring-triage`**](https://github.com/86sunbot/caldera-monitoring-triage) · [Draft PR #1](https://github.com/86sunbot/caldera-monitoring-triage/pull/1)).

---

## 4. AI Security Lab (`ai-security-lab`)

- **Visitor Access:** **Public case study with implementation currently private** (`86sunbot/ai-security-lab` is currently private and scheduled for Stage C public release).

### Problem and Audience
- **Problem:** Enterprise teams adopting LLMs and Retrieval-Augmented Generation (RAG) on Azure frequently overlook indirect prompt injection via retrieved documents, cross-tenant vector-index leakage, and excessive tool-execution permissions.
- **Audience:** AI Security Architects, Red Teamers, and Cloud Security Engineers securing Azure OpenAI / Microsoft Foundry workloads.

### Current Capabilities (Documented in Case Study)
- **LLM Application Threat Model (`threat-models/llm-threat-model.md`):** End-to-end decomposition of an enterprise LLM application across 5 trust boundaries using **STRIDE**, **OWASP Top 10 for LLM Applications**, and **MITRE ATLAS**, complete with risk scoring and control mappings.
- **Prompt Injection Attack Taxonomy & Defenses (`prompt-injection/attack-taxonomy.md`):** Classification of direct vs. indirect prompt injection, Crescendo multi-turn escalation, payload splitting, encoding bypasses, and multi-layer defenses (Azure AI Content Safety Prompt Shields, spotlighting/datamarking, output schema validation, and least-privilege tool gates).
- **Secure RAG Pipeline Architecture (`rag-security/rag-pipeline-security.md`):** Reference architecture for enterprise RAG on Azure covering document ingestion sanitization, Purview sensitivity-label inheritance, Azure AI Search document-level ACL security trimming, Private Link network isolation, and output grounding checks.

### Maturity / Status
- **Reference Architecture & Threat-Modeling Documentation (Existing Root `MIT` License Preserved):** Technical documentation repository (3 architecture documents).

### Evidence and Limitations
- **Validation Scope & Documented Pattern Scan:** No automated test suite is defined in this documentation repository; embedded Python snippets were syntax-checked via `ast.parse`, and regex pattern scans across `HEAD` and Git history detected no credentials or personal Azure GUIDs.
- **Known Limitations:** Documentation and reference patterns only; for an executable RAG implementation with public source and automated tests, see [`fde-lab/document-assistant/versions/v3-enterprise-assistant`](security-architecture.md#public-flagship-example-enterprise-aidr-architecture-comparison--rag-security-boundary-fde-lab).

### Canonical Source & Access Status
- **Access Status:** **Public case study with implementation currently private** (`86sunbot/ai-security-lab`, prepared on local branch `portfolio/remediation-2026-10` at commit `d9a4f2a` — Stage C candidate).
