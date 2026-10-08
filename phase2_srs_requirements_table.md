# Phase 2 — Requirements Engineering Deliverable
## SecureChain — Software Supply Chain Security Platform

---

## 1. Stakeholder & User Type Identification

The platform addresses learning, administrative, operational, and security analysis workflows across four primary actors:

| Actor / Stakeholder | Role & Responsibilities in System | Justification |
|---|---|---|
| **Student (Developer)** | • Registers project repositories and dependency manifests.<br>• Models CI/CD build pipelines and triggers vulnerability scans.<br>• Investigates identified security threats and executes remediations. | Primary target learner simulating hands-on software supply chain security workflows in an educational / lab context. |
| **Faculty (Instructor)** | • Reviews student project submissions and risk reports.<br>• Audits vulnerability findings and threat detection accuracy.<br>• Assigns lab benchmarks and evaluates remediation performance. | Oversees educational grading, validates lab exercises, and assesses student comprehension of supply chain risks. |
| **System Administrator** | • Manages user access controls, database backups, and API services.<br>• Monitors system uptime, API rate limits, and server logs.<br>• Updates threat intelligence feeds and package registries. | Ensures platform infrastructure health, security compliance, and system availability. |
| **Security Analyst (AppSec)** | • Configures custom threat detection rules & malicious package signatures.<br>• Conducts SLSA provenance audits and artifact checksum verification.<br>• Signs off on critical risk exceptions and compliance reports. | Expert operational actor representing enterprise AppSec teams validating real-world threat vectors. |

---

## 2. Security Architecture Mapping (CIA, AAA Controls)

To satisfy enterprise and educational security standards, requirements are mapped across security domains:

### CIA Triad Mapping
* **Confidentiality (C)**: Sensitive build secrets, API tokens, internal repository URLs, and student lab credentials must be protected against unauthorized disclosure via log redaction and encrypted transmission.
* **Integrity (I)**: Dependency lockfiles, build artifact SHA-256 digests, SLSA provenance data, and risk scores must be protected against unauthorized modification or tampering.
* **Availability (A)**: The scanner engine and API services must maintain high uptime and handle asynchronous concurrent scan processing without service disruption.

### AAA Controls (Authentication, Authorization, Audit)
* **Authentication (AuthN)**: JWT-based stateless authentication with password hashing (`bcrypt` / `passlib`) enforcing secure user session verification.
* **Authorization (AuthZ)**: Role-Based Access Control (RBAC) restricting administrative tasks (threat intel modification, user deletion) to Administrators/Faculty.
* **Audit (Auditability)**: Complete append-only audit trails for scan triggers, artifact verification events, status changes, and remediation resolutions.

---

## 3. Concise SRS / Requirements Table (Deliverable)

> [!NOTE]
> **Priority Scale (MoSCoW):**  
> **Must Have (M)** — Essential core feature for minimal viable platform.  
> **Should Have (S)** — High value feature necessary for full workflow modeling.  
> **Could Have (C)** — Desirable enhancement for advanced simulation.  
> **Won't Have (W)** — Out of scope for current iteration.

| Req ID | Requirement Description | Category | Priority | CIA Mapping | Security Control (AuthN / AuthZ / Audit) |
|---|---|---|---|---|---|
| **REQ-FR-01** | Register project metadata (Repo URL, Ecosystem, Owner, Team, Default Branch). | Functional | **Must Have** | Integrity | AuthN: Required<br>AuthZ: Student, Admin |
| **REQ-FR-02** | Import and parse direct & transitive dependency lockfiles (`package-lock.json`, `requirements.txt`). | Functional | **Must Have** | Integrity | AuthN: Required<br>Audit: Dep creation logged |
| **REQ-FR-03** | Detect **Dependency Confusion Attacks** by flagging internal package names resolving to public registries. | Security / Func | **Must Have** | Integrity | Audit: Finding logged with evidence |
| **REQ-FR-04** | Identify **Malicious Packages** by matching dependencies against threat intel feeds (`event-stream`, `xz-utils`). | Security / Func | **Must Have** | Integrity | Audit: Matched intel signature ID logged |
| **REQ-FR-05** | Detect **Artifact Tampering** by verifying SHA-256 digests against registered baseline digests. | Security / Func | **Must Have** | Integrity | Audit: Mismatch marks artifact `tampered=true` |
| **REQ-FR-06** | Identify **Build Compromise** by validating SLSA provenance metadata and build logs. | Security / Func | **Should Have** | Integrity | Audit: Provenance missing alert logged |
| **REQ-FR-07** | Scan build logs for **Secret Leakage** (AWS keys, GitHub PATs, Private Keys, Passwords). | Security / Func | **Must Have** | Confidentiality | Audit: Secret type & line snippet logged (redacted) |
| **REQ-FR-08** | Analyze **Transitive Graph Depth** and alert when dependency graph depth $\ge 5$. | Security / Func | **Should Have** | Integrity | Audit: Deep dependency IDs recorded |
| **REQ-FR-09** | Create, assign, and update **Remediation Tasks** through lifecycle states (`Open` $\rightarrow$ `Resolved`). | Functional | **Must Have** | Integrity | AuthZ: Student, Faculty<br>Audit: Resolution timestamp recorded |
| **REQ-FR-10** | Generate **CycloneDX v1.4 SBOM** and project Risk Report ($0.0 - 100.0$ score). | Functional | **Must Have** | Confidentiality / Integrity | AuthN: Required<br>Audit: Report generation logged |
| **REQ-NFR-01** | **Performance**: Complete security scan for 500 dependencies within $< 3\text{ seconds}$. | Non-Functional | **Should Have** | Availability | N/A |
| **REQ-NFR-02** | **Transport Security**: Encrypt all client-server communication using TLS 1.3 / HTTPS. | Security / NFR | **Must Have** | Confidentiality | AuthN: TLS handshake verification |
| **REQ-NFR-03** | **Role-Based Access Control**: Restrict Threat Intel editing to Administrator and Faculty roles. | Security / NFR | **Must Have** | Confidentiality / Integrity | AuthZ: Admin/Faculty role check enforced |
| **REQ-NFR-04** | **Audit Trail**: Maintain immutable timestamped log entries for all security scan executions. | Security / NFR | **Must Have** | Integrity | Audit: System log event persisted |
