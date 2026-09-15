# SOC 2 Readiness Control Matrix

Status values are `planned`, `partially_designed`, `implemented_not_evidenced`, `operating`, and `exception`. Nothing in this file is a certification claim. Human owners remain deliberately unassigned until PocketRankings.com LLC names them.

| ID | Area | Control objective | Frequency | Owner | Minimum evidence | Current status |
|---|---|---|---|---|---|---|
| GOV-01 | Governance | Approve security scope, policies, risks, and control ownership | Annual/change | Unassigned | Dated approval, scope, policy versions | Partially designed |
| IAM-01 | Access | Named accounts, MFA, least privilege, and approved access | Continuous | Unassigned | Access request, MFA state, role grant | Planned |
| IAM-02 | Access | Review Production, source, cloud, and vendor access | Quarterly | Unassigned | Export, reviewer, removals, closure | Planned |
| CHG-01 | Change | Reviewed branch/PR, tests, approval, rollback, exact deployment | Per change | Unassigned | Issue/PR, CI, approval, commit/image, deployment | Partially designed |
| VUL-01 | Vulnerability | Scan dependencies/configuration and remediate to approved deadlines | Per change/monthly | Unassigned | Scan, severity, owner, due date, closure | Partially designed |
| SEC-01 | Secrets | Store, restrict, rotate, and revoke secrets without logging them | Continuous/quarterly | Unassigned | Inventory metadata, access, rotation evidence | Planned |
| LOG-01 | Monitoring | Centralize redacted security/audit events and test actionable alerts | Continuous/quarterly | Unassigned | Log sample, alert test, investigation | Planned |
| AVL-01 | Availability | Monitor agreed service objectives and resolve breaches | Continuous/monthly | Unassigned | SLO report, incident/corrective action | Planned |
| BCP-01 | Recovery | Protected backups meet approved RPO/RTO and restore successfully | Scheduled/quarterly | Unassigned | Backup report, restore exercise, measured RPO/RTO | Planned |
| IR-01 | Incident | Detect, classify, contain, communicate, recover, and review incidents | Per incident/annual exercise | Unassigned | Incident or tabletop record and actions | Planned |
| VEN-01 | Vendors | Review security, availability, confidentiality, terms, and exit | Before use/annual | Unassigned | Due diligence, contract, review, deletion evidence | Planned |
| DAT-01 | Lifecycle | Classify/minimize data and execute approved retention/deletion | Continuous | Unassigned | Inventory, request/deletion receipts, exceptions | Partially designed |
| PRV-01 | Privacy | Authenticate, execute, and evidence irreversible player opt-out | Per request | Unassigned | Request, target receipts, completion/failure | Implemented not evidenced end-to-end |
| PER-01 | Personnel | Onboard, train, change, and promptly remove personnel access | Per event/annual | Unassigned | Checklist, training, access closure | Planned |

Before a Type I readiness review, every in-scope row needs a named owner, approved implementation description, evidence location, and resolved design exceptions.
