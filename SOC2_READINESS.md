# SOC 2 Readiness Plan

Last approved direction: 2026-09-15

## Target

Prepare first for a SOC 2 Type I examination covering Security, Availability,
and Confidentiality. After controls have operated for an auditor-selected
period, prepare for Type II. Privacy and Processing Integrity remain design
requirements where the services process personal data or official results.

Only an independent licensed CPA firm can issue a SOC 2 report. This plan
organizes readiness work and evidence; it is not a certification claim.

## Readiness workstreams

| Workstream | Required operating evidence |
|---|---|
| Governance and risk | Approved policies, system scope, risk register, control owners, annual and change-triggered reviews |
| Asset and data management | Product/service inventory, data flows, classifications, retention schedule, ownership, disposal evidence |
| Access control | Named identities, MFA, least privilege, access approvals, periodic reviews, termination evidence, emergency-access review |
| Secure change management | Request, branch, review, tests, approval, commit, deployment record, rollback, emergency retrospective |
| Vulnerability management | Scan reports, severity and owner, remediation date, verified closure, approved expiring exceptions |
| Infrastructure security | Hardened configuration, network isolation, encryption, secret inventory/rotation, patch and configuration evidence |
| Logging and monitoring | Time-synchronized audit/security events, redaction, alert ownership, alert tests, investigation and closure records |
| Availability and recovery | Approved availability objective, RTO/RPO, protected backups, restore tests, disaster-recovery exercises, corrective actions |
| Incident response | Approved plan, roles, escalation, evidence preservation, communication criteria, exercises and post-incident actions |
| Vendor management | Due diligence, contracts, data access, subprocessors, incident terms, periodic review, exit/deletion evidence |
| Personnel controls | Training, confidentiality obligations, onboarding, role changes, offboarding, policy acknowledgment |
| Privacy and confidentiality | Notices, purpose limitation, data minimization, rights requests, deletion/anonymization, legal holds, disposal evidence |

## Evidence rules

- Each control has a named human owner and an evidence location controlled by
  PocketRankings.com LLC.
- Evidence contains the minimum information necessary and never includes raw
  secrets or avoidable personal data.
- Automated reports identify the exact commit, image, environment, time, and
  result. Changed candidates require new evidence.
- Failed controls and exceptions remain visible until resolved; evidence is
  never rewritten to hide a failure.
- The evidence-retention period and audit observation window will be finalized
  with the selected CPA firm before the formal period begins.

## Before the Type I readiness review

1. Select an independent CPA firm and confirm scope, system boundary, criteria,
   subservice organizations, complementary user controls, and evidence period.
2. Approve the complete policy set and control matrix.
3. Assign control owners and train everyone with Production access.
4. Remediate design gaps and run one complete internal evidence review.
5. Conduct access review, vulnerability review, incident tabletop, backup
   restoration, disaster-recovery exercise, and vendor review.
6. Freeze the described system boundary and reconcile material changes with
   the auditor before the examination.
