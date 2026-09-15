# Risk Register

Likelihood and impact use `low`, `medium`, or `high`. Owners and treatment dates must be assigned by PocketRankings.com LLC; the entries below are an initial engineering inventory, not a completed management risk assessment.

| ID | Risk | Likelihood | Impact | Current treatment | Remaining action |
|---|---|---|---|---|---|
| R-01 | Account or product administrative compromise | Medium | High | Short sessions, secure cookies, role checks | MFA, recovery, access reviews, alerting |
| R-02 | Cross-product identity is linked to the wrong player | Medium | High | No name matching; explicit `PersonId` design | Signed handoff, replay protection, integration tests |
| R-03 | Privacy request is partially completed | Medium | High | Durable targets, idempotent product operations | Approved destinations, dispatcher, acknowledgements, alerts |
| R-04 | Restored backup resurrects deleted personal data | Medium | High | Replay requirement documented | Automated restore quarantine and ledger replay test |
| R-05 | Customer stack is not deleted after 61 days | Medium | High | Lifecycle state machine designed | Durable scheduler, deletion executor, evidence, failure escalation |
| R-06 | One customer can access another customer's stack/data | Low | High | Per-installation database/network design | Provisioning integration and isolation negative tests |
| R-07 | Unpatched dependency or host vulnerability is exploited | Medium | High | NuGet scans in product validation | Host/container scanning, remediation SLA, exception process |
| R-08 | Data loss or prolonged outage | Medium | High | Recovery requirements documented | Approve SLO/RPO/RTO, backups, restore/DR exercises |
| R-09 | Secret exposure in source, logs, or artifacts | Medium | High | Repository rules and placeholder configs | Managed secret store, scanning, rotation evidence |
| R-10 | Third-party social embeds create privacy/licensing exposure | Medium | Medium | Visitor initiated; URL-only storage | Current terms/privacy review and launch decision |
| R-11 | Unsupported tournament format produces incorrect results | Low | High | Swiss/group-to-finals fail closed | Implement and fully simulate before enablement |
| R-12 | Compliance readiness is mistaken for certification | Low | High | Explicit non-certification language | Formal scope review and licensed CPA engagement |
