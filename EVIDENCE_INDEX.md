# Control Evidence Index

The authoritative evidence store has not been selected. When selected, it must be private, access-controlled, backed up, and owned by PocketRankings.com LLC. Do not place secrets or unnecessary personal data in evidence.

| Evidence family | Required metadata | Suggested cadence | Current source |
|---|---|---|---|
| Change and release | Request, approval, commit, image digest, tests, environment, rollback | Per deployment | GitHub and product handoff docs |
| Access review | System, named account, role, MFA, approver, disposition | Quarterly | Not established |
| Vulnerability | Exact target, scanner/source, findings, severity, due/closed dates | Per change/monthly | Product dependency scan output |
| Backup and restore | Dataset classification, backup ID, encryption/access, restore result, measured RPO/RTO | Scheduled/quarterly | Not established |
| Monitoring and alerts | Rule/version, test event, recipient, response, closure | Quarterly/change | Not established |
| Incident response | Severity, timeline, decisions, communications, evidence, corrective actions | Per incident/annual exercise | Not established |
| Privacy request | Request ID, authenticated confirmation, target receipts, completion/failure, no raw token | Per request | Account/product privacy ledgers once connected |
| Customer deletion | Installation ID, cancellation, deadline, hold decision, deleted resources/backups, verifier | Per deletion | Lifecycle foundation; executor not established |
| Vendor review | Service, data/access, terms, assurance, risks, approval, exit/deletion | Before use/annual | Not established |
| Training/personnel | Person, role, required training, acknowledgment, access changes | Onboarding/annual/offboarding | Not established |

Evidence must be immutable or versioned, time synchronized, attributable, minimally necessary, and linked to the exact control and system state it supports.
