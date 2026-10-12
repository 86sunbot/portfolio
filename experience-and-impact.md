# Experience, Credentials & Verified Impact

[**← Back to Portfolio Index**](README.md) · [**Security Architecture →**](security-architecture.md) · [**Microsoft Sentinel →**](sentinel.md) · [**AI Agents →**](ai-agents.md) · [**Books & Learning →**](books-and-learning.md)

---

## Professional Summary

**Sharat P Surya (`Surya PS`), CISSP**  
*Cybersecurity Architect · Manager 2 – Cybersecurity at EY GDS · Bengaluru, India*

Security Architect and cybersecurity engineering leader translating business, regulatory, and operational requirements into **Azure and hybrid enterprise security architectures**, **Microsoft Sentinel & Defender XDR detection/SOAR programs**, and **governed AI security engineering workflows**.

---

## Career Roles & Practice Areas

> **Stage A Scope Note:** Employment roles below summarize my core practice areas across EY GDS, Accenture, and Cognizant. Quantitative client delivery metrics, internal sub-role date boundaries, and employer recognition awards from my curriculum vitae are omitted from this Stage A public catalogue pending owner confirmation. For independently verifiable public evidence, see the **ISC2 CISSP Credly verification link** and the **GitHub portfolio entries** mapped below.

### 1. EY GDS — Manager 2 – Cybersecurity
**Bengaluru, India**

- Lead Microsoft Security practice initiatives connecting security architecture, capability development, engineering standards, reusable solution patterns, and delivery quality.
- Provide technical direction across **Microsoft Sentinel**, **Microsoft Defender XDR**, detection engineering, security automation, and AI-assisted SecOps.
- Translate enterprise security requirements into actionable architecture and engineering approaches across identity, cloud, and SIEM domains.
- Design and build governed AI-assisted Microsoft Sentinel detection engineering workflows covering workspace telemetry validation, KQL rule generation, static quality checks, human review, and `enabled=false` deployment gates.

---

### 2. Accenture — Cloud Security Architecture & SecOps
**Bengaluru, India**

- **Enterprise Cloud & Hybrid Architecture:** Architected Azure and hybrid cloud-security solutions, translating business, infrastructure, and compliance requirements into **High-Level Designs (HLDs)**, **Low-Level Designs (LLDs)**, and phased implementation plans.
- **Reusable Architecture Governance:** Authored reusable architecture frameworks, cloud/network/SIEM security assessment methodologies, implementation checklists, and handover governance structures.
- **SIEM & Microsoft Defender Engineering:** Designed log-source onboarding strategies, telemetry requirements, MITRE ATT&CK-aligned KQL detections, and Azure Logic Apps SOAR playbooks across **Microsoft Sentinel**, **Microsoft Defender for Endpoint (MDE)**, **Microsoft Defender for Identity (MDI)**, and **Microsoft Defender XDR**.
- **Azure Network Security & Automation:** Designed Azure network-security architectures covering **Palo Alto Networks / Panorama high availability**, **IPsec VPN**, **BGP-based hybrid connectivity**, and network segmentation, paired with Python and PowerShell operational automation.

---

### 3. Cognizant Technology Solutions — Network & Infrastructure Security
**Bengaluru, India**

- Conducted enterprise network-security assessments and L3 security operations across **FortiGate**, **Cisco ASA**, and **SonicWall** firewall estates, including VPN troubleshooting, firewall policy governance, and standardized operating procedures (SOPs).

---

## Verified Credentials & Education

### Active Publicly Verified Certification

| Certification | Issuing Body | Status & Independent Verification Link |
| :--- | :--- | :--- |
| **CISSP — Certified Information Systems Security Professional** | **ISC2** | **Active & Publicly Verified** · Issued: `16 January 2026` · Expires: `31 January 2029`<br>[**Verify ISC2 Credly Badge →**](https://www.credly.com/badges/5c478c9e-77f0-4227-9251-69ecb1d639e5) |

*(Note: Historical Microsoft [`SC-200`, `AZ-500`] and network vendor [`PCNSE`, `NSE 4`, `CCNA Security`] certifications held between 2018 and 2023 are expired on Credly and are omitted from active credential claims).*

### Education
- **Bachelor of Technology (B.Tech) in Computer Science & Engineering** — **Pondicherry University** (`2015`)

---

## How Practice Areas Map to Portfolio Evidence

| Practice Area | Portfolio Entry & Visitor Access |
| :--- | :--- |
| **Microsoft Sentinel Detection Engineering & AI SecOps** | • [`fde-lab/sentinel-usecase-deployment`](sentinel.md#public-flagship-example-entra-id-password-spray-kql-detection--two-layer-validator-fde-labsentinel-usecase-deployment) — **Public source and demo instructions:** Entra ID password spray KQL rule, two-layer validator, and governed agent workflow (`28 passed` `pytest` tests).<br>• [`ai-agent-usecase-deployment`](ai-agents.md#1-ai-assisted-microsoft-sentinel-detection-engineering-agent-ai-agent-usecase-deployment-v34) — **Public case study (implementation currently private):** V3.4 AI Detection Engineering Agent (`295 passed, 1 skipped` automated `pytest` tests).<br>• [`sentinel-detection-engineering`](sentinel.md#3-sentinel-detection-engineering-library-sentinel-detection-engineering) — **Public case study (implementation currently private):** 6 MITRE ATT&CK v14 KQL rules & methodology (`6/6` YAML schema checks passed).<br>• [`Sentinel-Log-Validation-Agent`](sentinel.md#1-sentinel-log-validation-agent-sentinel-log-validation-agent) — **Public case study (implementation currently private):** Deterministic onboarding assurance (`130 passed` automated `pytest` tests). |
| **Greenfield SIEM Provisioning & SOAR Automation** | • [`Sentinel-as-a-Service`](sentinel.md#2-sentinel-as-a-service--greenfield-deployment-kit--orchestrator-sentinel-as-a-service) — **Public case study (implementation currently private):** 10-stage web wizard + PowerShell 7 ARM REST orchestrator + Terraform greenfield module (`24/24` Pester tests passed).<br>• [`security-automation`](sentinel.md#4-security-operations-automation-toolkit-security-automation) — **Public case study (implementation currently private):** Managed Identity Logic App ARM playbook + PowerShell/Python/KQL toolkit (`6/6` Pester + `5/5` `pytest` passed). |
| **Azure & Hybrid Cloud Security Architecture** | • [`fde-lab`](security-architecture.md#public-flagship-example-enterprise-aidr-architecture-comparison--rag-security-boundary-fde-lab) — **Public source and demo instructions:** Enterprise AI/DR architecture comparison ([`microsoft_vs_crowdstrike_ai_dr_guide.md`](https://github.com/86sunbot/fde-lab/blob/main/microsoft_vs_crowdstrike_ai_dr_guide.md)) + `v3-enterprise-assistant` security boundary (`32 passed` `pytest` tests).<br>• [`azure-security-framework`](security-architecture.md#1-azure-security-architecture-framework-azure-security-framework) — **Public case study (implementation currently private):** CAF Landing Zones, Conditional Access, Hub-Spoke, Encryption, and Azure Policy.<br>• [*Microsoft Security Architect Master Blueprint v1.1*](books-and-learning.md#1-microsoft-security-architect-master-study-guide--reference-architecture-v11) (958 pp.) & [*SC-500 Complete Study Guide v1.1*](books-and-learning.md#2-microsoft-sc-500-cloud-and-ai-security-engineer--complete-study-guide-v11) (434 pp.) — **Public chapter outlines (manuscripts currently private).** |
| **Forward Deployed Engineering (FDE) & Bounded AI Systems** | • [`FDE-Final-Capstone`](https://github.com/86sunbot/FDE-Final-Capstone) — **Public source and demo instructions:** Fail-closed state machine & tamper-evident hash chain (`118 passed` automated `pytest` tests, `96%` coverage).<br>• [`caldera-monitoring-triage`](https://github.com/86sunbot/caldera-monitoring-triage) — **Public source and demo instructions:** Pre-GxP report surfacing ([Sprint Walkthrough](https://github.com/86sunbot/caldera-monitoring-triage/blob/main/docs/FDE_SPRINT_STORY_AND_WALKTHROUGH.md) & [Demo Script](https://github.com/86sunbot/caldera-monitoring-triage/blob/main/docs/DEMO_PRESENTATION_SCRIPT.md); `9 passed` automated `pytest` tests on public `main`, `10` on [Draft PR #1](https://github.com/86sunbot/caldera-monitoring-triage/pull/1)).<br>• [`fde-lab`](https://github.com/86sunbot/fde-lab) — **Public source and demo instructions:** AI/DR Architecture Comparison + `60 passed` automated `pytest` tests across v3 assistant & Sentinel v1. |
| **CISSP Mastery & Security Governance** | • [Public Cross-Domain Sample Lesson](books-and-learning.md#publicly-inspectable-sample-lesson-cross-domain-architecture-walk-through-cissp--entra-id-sc-100sc-500--microsoft-sentinel) + [*Surya Master CISSP Guide*](books-and-learning.md#3-surya-master-cissp-guide-eight-domain-mindmaps--cissp-knowledge-base) (382 pp.) & `cissp-knowledge-base` — **Public sample lesson & domain outlines (manuscript and repo currently private).** |
