# Asset And Vendor Register

This is the initial SOC 2 readiness inventory. `TBD` means a human selection or verified Production fact is still required; it does not mean the control operates.

| Asset/service | Owner | Classification | Purpose | Environment/data access | Backup/exit evidence | Review status |
|---|---|---|---|---|---|---|
| PocketRankingsAccount | Unassigned | Confidential | Identity, authentication, entitlements, lifecycle/privacy orchestration | Production TBD; identity and purchase metadata | TBD | Design recorded |
| PocketRankingsPlayerProfile | Unassigned | Confidential | Shared player profile and derived player-level history | Production TBD; player personal data | TBD | Design recorded |
| PocketRankingsTournament installations | Unassigned | Confidential | Isolated customer tournament operations | One stack/database per customer | 61-day deletion design; restore evidence TBD | Design recorded |
| PoolLeagueWeb installations | Unassigned | Confidential | Isolated customer league operations | Separately owned/operated | 61-day deletion design; evidence TBD | Not assessed here |
| PocketRankingsWeb | Unassigned | Public/Internal | Marketing, pricing, future checkout entry | Production TBD | TBD | Design recorded |
| Source control/CI provider | Unassigned | Confidential | Code, review, build evidence | Provider TBD | Export/offboarding TBD | Selection required |
| Hosting, database, secrets, logging, backup, payment, email, and DNS providers | Unassigned | Varies | Production supporting services | Providers TBD | Contract, attestations, deletion/exit TBD | Selection required |

Before use, each vendor record must include legal entity, service owner, data categories/locations, subprocessors, authentication/MFA, encryption, availability, incident notice, retention/deletion, contract/DPA, current assurance report where applicable, renewal date, and exit test. Never place credentials or personal data in this register.
