# Phase 2 — Requirements Engineering Deliverable
## SecureChain — Software Supply Chain Security Platform

---

## 1. Stakeholder & User Type Identification

The platform addresses learning, administrative, operational, and security analysis workflows across four primary stakeholders:

| Stakeholder / User Type | Role & Responsibilities in System | Justification |
|---|---|---|
| **Developer** | • Registers projects, manages dependencies, initiates scans, views vulnerabilities and tracks remediation.<br>• Performs daily software development and security remediation tasks.<br>• Primary end-user of the platform. | Core primary actor modeling software supply chain security workflows in development. |
| **Security Administrator** | • Monitors security findings, reviews risk reports, manages security policies and verifies remediation.<br>• Audits vulnerability findings and threat intelligence feeds.<br>• Signs off on compliance reports and risk exceptions. | Responsible for security governance, vulnerability sign-offs, and risk verification. |
| **Project Administrator** | • Manages projects, project metadata, dependencies and build artifacts.<br>• Configures team ownership, default branches, and repository mappings.<br>• Audits artifact SHA-256 digests and build provenance. | Manages project lifecycle, build configurations, and component inventories. |
| **System Administrator** | • Maintains the platform, database, backend services and access permissions.<br>• Manages infrastructure uptime, system backups, API gateways, and user accounts.<br>• Enforces Role-Based Access Control (RBAC). | Ensures platform infrastructure health, security compliance, and system availability. |

---

## 2. Security Architecture Mapping (CIA, AAA Controls)

### CIA Triad Mapping
* **Confidentiality (C)**: Sensitive build secrets, API tokens, internal repository URLs, and user credentials are protected via JWT tokens, bcrypt salted hashes, and regex log redaction.
* **Integrity (I)**: Lockfile manifests, artifact SHA-256 digests, SLSA provenance data, and risk scores are protected against unauthorized modification or tampering.
* **Availability (A)**: The scanner engine and API services execute asynchronously to maintain low API latencies and high service availability.

### AAA Controls (Authentication, Authorization, Audit)
* **Authentication (AuthN)**: JWT-based stateless authentication with password hashing enforcing user session verification (`/auth/login`).
* **Authorization (AuthZ)**: Role-Based Access Control (RBAC) enforcing role permissions (`Developer`, `Security Administrator`, `Project Administrator`, `System Administrator`).
* **Audit (Auditability)**: Complete append-only audit trails for scan triggers, artifact verification events, status changes, and remediation resolutions.

---

## 3. Concise SRS / Requirements Table (Deliverable)

| Req ID | Requirement Description | Category | Priority | CIA Mapping | Security Control (AuthN / AuthZ / Audit) |
|---|---|---|---|---|---|
| **REQ-FR-01** | Register project metadata (Repo URL, Ecosystem, Owner, Team, Default Branch). | Functional | **Must Have** | Integrity | AuthN: Required<br>AuthZ: Developer, Project Admin |
| **REQ-FR-02** | Import direct & transitive dependency lockfiles (`package-lock.json`, `requirements.txt`). | Functional | **Must Have** | Integrity | AuthN: Required<br>Audit: Dep creation logged |
| **REQ-FR-03** | Detect **Dependency Confusion Attacks** by flagging internal package names resolving to public registries. | Security / Func | **Must Have** | Integrity | Audit: Finding logged with evidence |
| **REQ-FR-04** | Identify **Malicious Packages** by matching dependencies against threat intel feeds (`event-stream`, `xz-utils`). | Security / Func | **Must Have** | Integrity | Audit: Matched intel signature ID logged |
| **REQ-FR-05** | Detect **Artifact Tampering** by verifying SHA-256 digests against registered baseline digests. | Security / Func | **Must Have** | Integrity | Audit: Mismatch marks artifact `tampered=true` |
| **REQ-FR-06** | Identify **Build Compromise** by validating SLSA provenance metadata and build logs. | Security / Func | **Should Have** | Integrity | Audit: Provenance missing alert logged |
| **REQ-FR-07** | Scan build logs for **Secret Leakage** (AWS keys, GitHub PATs, Private Keys, Passwords). | Security / Func | **Must Have** | Confidentiality | Audit: Secret type & line snippet logged (redacted) |
| **REQ-FR-08** | Analyze **Transitive Graph Depth** and alert when dependency graph depth $\ge 5$. | Security / Func | **Should Have** | Integrity | Audit: Deep dependency IDs recorded |
| **REQ-FR-09** | Create, assign, and update **Remediation Tasks** through lifecycle states (`Open` $\rightarrow$ `Resolved`). | Functional | **Must Have** | Integrity | AuthZ: Developer, Security Admin<br>Audit: Resolution timestamp recorded |
| **REQ-FR-10** | Generate **CycloneDX v1.4 SBOM** and project Risk Report ($0.0 - 100.0$ score). | Functional | **Must Have** | Confidentiality / Integrity | AuthN: Required<br>Audit: Report generation logged |
| **REQ-NFR-01** | **Performance**: Complete security scan for 500 dependencies within $< 3\text{ seconds}$. | Non-Functional | **Should Have** | Availability | N/A |
| **REQ-NFR-02** | **Transport Security**: Encrypt all client-server communication using TLS 1.3 / HTTPS. | Security / NFR | **Must Have** | Confidentiality | AuthN: TLS handshake verification |
| **REQ-NFR-03** | **Role-Based Access Control**: Enforce authorization checks for Developer, Security Admin, Project Admin, System Admin roles. | Security / NFR | **Must Have** | Confidentiality / Integrity | AuthZ: RBAC role check enforced |
| **REQ-NFR-04** | **Audit Trail**: Maintain immutable timestamped log entries for all security scan executions. | Security / NFR | **Must Have** | Integrity | Audit: System log event persisted |
