# Automated Customer Provisioning

Last approved direction: 2026-09-15

## Deployment model

Use a shared managed host pool initially, with an isolated Docker Compose
project and PostgreSQL database for each purchased League or Tournament
customer product. Reuse the same immutable, validated product image; never
create customer-specific source branches or builds.

Keep purchase orchestration independent of the deployment executor so Compose
can later be replaced by a managed container orchestrator without rewriting
checkout or entitlement logic.

## State machine

`purchase_confirmed → entitlement_recorded → provisioning_queued → infrastructure_created → migrating → validating → ready`

Failures enter a bounded retry or `needs_attention` state. Cancellation enters
`access_disabled → retention_61_days → deletion_due → deleting → deleted`.

## Required controls

- Signed and idempotent payment webhook handling
- Separate purchase and entitlement for League and Tournament
- Durable provisioning queue and request identifier
- Customer-safe slug and hostname reservation
- Per-installation credentials from an approved secret store
- Isolated stack, database, storage, network, and backups
- Idempotent migrations with protected recovery evidence
- Single-use first-administrator handoff
- Health, schema, isolation, version, and permission checks before readiness
- Customer-visible setup status and owner-visible failure alerting
- Complete audit correlation from purchase through deletion
- No real provider, DNS, secret, or Production activation without its separate
  approval and launch gate
