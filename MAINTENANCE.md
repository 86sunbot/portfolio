# Portfolio Governance, Source-of-Truth Model & Maintenance Guide

[**← Back to Portfolio Index**](README.md)

This document defines the source-of-truth architecture, maturity taxonomy, routing rules, and pre-publication procedures governing `86sunbot/86sunbot` (Profile README), `86sunbot/portfolio` (Portfolio Catalogue), and individual project repositories.

---

## 1. Three-Layer Source-of-Truth Model

To prevent duplicated or drifting claims across repositories, public work is organized into three layers:

| Layer | Location | Role |
| :--- | :--- | :--- |
| **Layer 1 — Public Introduction** | [`86sunbot/86sunbot`](https://github.com/86sunbot/86sunbot) | Concise ~450-word introduction, 3 featured examples, 5 category links, books summary, and verified credentials. |
| **Layer 2 — Portfolio Catalogue (This Repo)** | `86sunbot/portfolio` | Authoritative index of projects, architecture patterns, books, maturity classifications, test/page metrics, and career context. |
| **Layer 3 — Project Repositories** | Individual repositories | Canonical source for executable code, tests, CI workflows, schemas, and release artifacts. |

---

## 2. Evidence & Maturity Taxonomy

Every project in this catalogue is assigned an explicit maturity classification on its category detail page:

- **Enterprise Practice Role Summary:** Qualitative career role descriptions (documented in [`experience-and-impact.md`](experience-and-impact.md); unconfirmed quantitative client metrics and proprietary client artifacts are excluded).
- **Working Engineering Prototype (Locally Tested):** Executable software or Infrastructure-as-Code with automated unit/integration test suites and deterministic verification harnesses; explicitly distinguishes *defined* tests from *locally executed* tests, *schema validation* from *live query execution*, and **public source and demo instructions** from **public case studies with implementation currently private**.
- **Academic / Sprint Simulation (Locally Tested):** Systems built against synthetic enterprise datasets under structured Forward Deployed Engineering (FDE) stage gates; explicitly non-production and non-clinical.
- **Reference Architecture & Unpublished Self-Authored Manuscript:** Self-authored book manuscripts (local PDF/Markdown), companion study repositories, KQL/policy libraries, and threat-modeling blueprints.

---

## 3. Where New Content Belongs (Routing Rules)

Every piece of work has **exactly one canonical home**. Never paste duplicate code, full architecture guides, or duplicate test tables across multiple repositories.

| Content Type | Canonical Home (Layer 3) | Catalogue Summary (Layer 2: `portfolio/`) | Profile Summary (Layer 1: `86sunbot/86sunbot`) |
| :--- | :--- | :--- | :--- |
| **Azure / Zero Trust Reference Designs, Bicep Policies, HLD/ADR Templates** | `azure-security-framework` *(Private — Stage B)* & `fde-lab/microsoft_vs_crowdstrike_ai_dr_guide.md` *(Public)* | Summarize & link in [`security-architecture.md`](security-architecture.md) | Featured Example #3 & category link |
| **KQL Detection Rules, MITRE Navigator Layers, Detection Quality Rubrics** | `sentinel-detection-engineering` *(Private — Stage B)* | Summarize & link in [`sentinel.md`](sentinel.md) | Featured Example #1 & category link |
| **Sentinel Log Onboarding Validation, Canary Tests, ASIM/FinOps Scoring** | `Sentinel-Log-Validation-Agent` *(Private — Stage B)* | Summarize & link in [`sentinel.md`](sentinel.md) | Featured Example #1 & category link |
| **Sentinel Greenfield Provisioning (PowerShell 7 / ARM / Wizard UI / Terraform)** | `Sentinel-as-a-Service` *(Private `origin/main` @ `44ca4b0`; consolidates `Terraform_Sentinel`)* | Summarize & link in [`sentinel.md`](sentinel.md) | Category link |
| **Sentinel SOAR Playbooks (Logic Apps), Entra ID / Azure / Network Audits & Utilities** | `security-automation` *(Private `origin/main` @ `f0c65c4`; consolidates `Sentinel`, `PowerShell_Automation`, `Palo-Alto-Codes`, `Python_Automation`)* | Summarize & link in [`sentinel.md`](sentinel.md) | Category link |
| **AI-Assisted Sentinel Detection Engineering Agent (V2–V3.4+)** | `ai-agent-usecase-deployment` *(Private — Stage C)*; V1 in `fde-lab` *(Public)* | Summarize & link in [`ai-agents.md`](ai-agents.md) (cross-link from `sentinel.md`) | Featured Example #1 |
| **LLM Threat Models, Prompt Injection Defenses, Secure RAG Blueprints** | `ai-security-lab` *(Private — Stage C)* | Summarize & link in [`ai-agents.md`](ai-agents.md) | Category link |
| **FDE Capstones & Applied Engineering Prototypes** | `FDE-Final-Capstone`, `caldera-monitoring-triage`, `fde-lab` *(All Public)* & `maritime-fde-workshop-demo` *(Private active workshop app)* | Summarize & link in [`ai-agents.md`](ai-agents.md) | Featured Example #2 (`FDE-Final-Capstone`) |
| **Technical Books, Certification Study Guides & CISSP Domain Notes** | `cissp-knowledge-base` *(Private — Stage C)* and local manuscript archives | Document exact title, edition, syllabus date, and page count in [`books-and-learning.md`](books-and-learning.md) | Short Books section |
| **Quantitative Trading Backtests & Market Research** | `trading-research` *(Private `origin/main` @ `f00ae2a`; consolidates algorithms from `Backtest_Codes`, `Functions`, `archive-python-utils`, `archive-trading-strategies`)* | Summarize & link in [`trading-research.md`](trading-research.md) | Category link #5 (separated from cybersecurity) |
| **Career Roles & Verified Certifications** | [`experience-and-impact.md`](experience-and-impact.md) | Canonical home for public career context | Credentials & Contact link |

---

## 4. Pre-Publication Safety & Verification Procedures

Before changing any private repository to Public (`gh repo edit 86sunbot/<repo> --visibility public`):

1. **Inspect Both `HEAD` and Full Git History (`git log --all -p`):**
   - Distinguish **authentication credentials/secrets** (passwords, PINs, API keys, bot tokens — which require immediate revocation/rotation and permanent private isolation) from **non-secret cloud/personal identifiers** (Azure Subscription/Tenant GUIDs, subscription display names, personal emails/hostnames — which require a clean-history public companion repository before public release).
   - Document exact regex/pattern scan scope rather than claiming absolute assurances.
2. **Mark Sanitized Sample Artifacts Explicitly:**
   - Any sample evidence JSON or verification report whose identifiers were scrubbed must include explicit provenance fields (`artifact_provenance: "SYNTHETIC_SANITIZED_SAMPLE"`, `sanitized_at`) so it is never presented as an unaltered raw cloud dump.
3. **Separate Defined vs. Executed Tests and Schema vs. Runtime Checks:**
   - Record which test suites were executed locally vs. defined for CI environments, and separate static YAML/KQL/Terraform schema checks from live cloud/Log Analytics execution.
4. **Preserve Existing Licenses & Leave New Licensing Choices Pending:**
   - Preserve existing repository licenses (`MIT` in `azure-security-framework`, `sentinel-detection-engineering`, `Sentinel-as-a-Service`, `security-automation`, `ai-security-lab`, and `cissp-knowledge-base`; `LICENSE-SYNTHETIC-TRAINING.txt` in `FDE-Final-Capstone/source_baseline`). Leave new licensing choices (`portfolio`, root `FDE-Final-Capstone`, `caldera-monitoring-triage`, `fde-lab`, `Sentinel-Log-Validation-Agent`, `ai-agent-usecase-deployment`, `trading-research`, and book companion materials) pending explicit owner selection. Do not treat exclusions from an `MIT` code license as authorization to redistribute third-party documentation fixtures or coursework files.
5. **Unauthenticated Link Verification:**
   - Confirm zero links in public files point to private repositories (`HTTP 404`) or unapproved book preview images.

---

## 5. Maintenance Cadence

- **Test & Evidence Updates:** Update detailed test counts, coverage metrics, and manuscript page counts on the relevant detail page (`sentinel.md`, `ai-agents.md`, `books-and-learning.md`) and in the project repository's own `README.md`, keeping the landing page (`README.md`) and profile (`86sunbot/README.md`) focused on examples and navigation.
- **Book & Certification Currency:** Record exam syllabus revisions (`SC-100`, `SC-500`, `CISSP`, `AI-901`) in [`books-and-learning.md`](books-and-learning.md) and verify Credly badge links annually.
