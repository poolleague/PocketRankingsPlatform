# Pocket Rankings Platform — Shared Working Rules

This document defines the rules shared by every product in the Pocket
Rankings platform: `PoolLeagueWeb` (League), `PocketRankingsAccount`
(Identity/Entitlements), `PocketRankingsWeb` (Marketing + Paywall),
`PocketRankingsTournament` (Tournament), and `PocketRankingsPlayerProfile`
(Professional Player).

Each product repository has its own local `AGENTS.md` covering what is
specific to that product. That file inherits everything here. Where a local
file and this file conflict, stop and ask the owner rather than silently
picking one — that conflict means something drifted and needs a decision,
not an assumption.

Status: DRAFT — pending owner review before any product repo formally adopts
it.

## 1. Product Boundaries And Isolation

- League, Tournament, Player Profile, Account, and the marketing Website are
  five separately deployed products. Each keeps its own repository, database,
  network, queues, configuration, backups, credentials, and operational
  state.
- Network isolation is provided by each product shipping its own
  `docker-compose.yml` in its own repository, deployed as its own Compose
  project. This is not extra configuration — it is the default behavior of
  running each product as a separate Compose project, and it falls directly
  out of each product already being a separate repository. A shared reverse
  proxy (Caddy) may join more than one product's network to route by
  hostname, matching the pattern PoolLeagueWeb already uses to route its own
  Production/SB/Demo environments by domain. Two products' backend
  containers must not be reachable from one another at the network layer
  absent an explicit, documented reason.
- No product's runtime code queries another product's database directly, and
  no two products share a database, connection string, or credential.
- A product must remain fully operational if every other product is down,
  except for the specific cross-product calls described in Section 2 (token
  verification, entitlement checks, event publication). Those calls must fail
  safely: a product should degrade gracefully (e.g., treat an unreachable
  Account service as "cannot confirm elevated access" rather than crashing or
  silently granting access).
- Never automatically copy contacts, consent, credentials, tokens, sessions,
  delivery history, or integration state between products. Any data that
  crosses a product boundary does so only through the identity/entitlement
  and event contracts defined below — never through direct database access,
  shared secrets, or ad hoc file transfer.

## 2. Shared Identity And Entitlements

### 2.1 Identity

- `PocketRankingsAccount` is the single source of truth for human identity
  across the platform. Every person has exactly one `PersonId` (GUID),
  issued and owned by Account.
- Each product keeps its own local accounts/roles table for its own
  in-product concerns (e.g., League's Admin/Officer/Team roles remain local
  to League and are not replaced by this system).
- When a person authenticates through Account, Account issues a short-lived
  signed token containing their `PersonId` and issued-at/expiry times.
  Products verify this token's signature locally (via a published public
  key) and do not need a live network call back to Account to validate a
  token's authenticity.
- On first sight of a given `PersonId`, a product creates a local link row
  (mirroring League's existing `IntegrationEntityLink` pattern, applied to a
  `person` entity type) associating that `PersonId` with the product's own
  local account record. This link is the only place a `PersonId` is stored
  outside Account itself.

### 2.2 Entitlements

- Account owns a single entitlements table: rows of
  `(PersonId, ProductType, Tier, Source, GrantedAt, ExpiresAt)`.
- Every product-side access check answers one question: "does this
  `PersonId` have a current, unexpired entitlement row for this product?"
  Products query this either via a signed claim embedded in the token or a
  narrow read-only entitlement-check call to Account — never by trusting
  client-supplied state.
- The entitlement check itself never changes based on business stage. What
  changes over time is only *how rows get created* (see Section 3). This
  keeps the enforcement code stable even as the business model evolves.

## 3. Current-Stage Entitlement Policy (Beta)

This section documents today's actual policy. Update it explicitly, with
owner approval, as the business stage changes — do not let product-side
enforcement code silently drift out of sync with what this section says.

- League, Tournament, and Player Profile are included at no charge for all
  current League beta customers for a minimum of six months from a given
  customer's initial paid League registration. No recurring income is
  expected from these three products during this window.
- Entitlement rows for included access are created automatically
  (`Source: "beta_included"`) at League registration, not through a
  checkout flow.
- Included-access rows use an explicit, clearly-labeled placeholder
  expiration far beyond any realistic near-term business decision (e.g. a
  constant such as `BetaAccessExpiresAt = 2099-01-01`), documented in code as
  a placeholder, never presented to a customer as a real commitment.
- The marketing Website's pricing/checkout UI is real, functioning UI wired
  to a real checkout code path — not a mockup — but that path currently
  terminates in a no-op payment provider (Section 4) rather than a live
  processor.

## 4. Payment Provider Abstraction

- All checkout flows in `PocketRankingsWeb` and Account call a payment
  provider through an interface (e.g. `IPaymentProvider`), never a concrete
  payment SDK directly.
- Today's registered implementation is a no-op/stub provider: it performs no
  real charge and, per Section 3, either grants beta-included access or
  clearly labels the flow as inactive.
- Moving to a live processor (e.g. Stripe) means implementing and swapping in
  one new class behind the existing interface. It must not require changes
  to entitlement-check logic in League, Tournament, or Player Profile.
- Activating a live payment provider, in any environment that can move real
  money, requires explicit owner approval and is treated with the same
  seriousness as a Production database change (Section 7).

## 5. Purchase Fulfillment Vs. Code Deployment

These are different kinds of "automation" and are governed differently.

### 5.1 Purchase fulfillment — autonomous

- Once a live payment provider is active, granting access after a successful
  payment must be fully automated: payment provider webhook → signature
  verification → entitlement row creation, with no manual step and no owner
  action required for the standard successful-payment case.
- Failed payments, disputes, chargebacks, and refund-triggered access
  revocation are exceptions that generate an owner-visible notification
  rather than being silently auto-resolved or silently ignored.

### 5.2 Code deployment — owner-gated

- Automated CI (build, automated tests, static checks) may run on every push
  without approval, matching League's existing practice.
- Promotion of any product to its Production environment always requires
  explicit owner approval before it happens, regardless of how much of the
  mechanical process (build image, run gate, push to host) is scripted or
  automated. A passing automated pipeline is evidence supporting a promotion
  decision, not a substitute for it.
- This mirrors League's existing SB-gate-then-owner-review-then-Production
  posture. New products should meet or exceed it, not relax it.

## 6. Coding And Repository Consistency

- All five products share one stack: ASP.NET Core MVC (C#), PostgreSQL,
  Docker, Caddy. A new product does not introduce a different backend
  framework or database engine without an explicit, owner-approved reason
  recorded in that product's local `AGENTS.md`.
- All product repos follow League's existing shape:
  `Controllers/ Services/ Models/ Views/ wwwroot/`, plus a `docs/` tree for
  release/deployment/testing documentation and a `CODE_OUTLINE.md`
  explaining where to make common changes.
- Shared, non-product-specific code — session/token verification, audit
  logging, the outbox/inbox event envelope, the entitlement-check client —
  lives in one internal shared library, versioned and referenced by every
  product repo, rather than being reimplemented per product. Changes to the
  shared library are reviewed with extra care since a defect there affects
  every product at once.
- A shared, lightweight design system (CSS variables, layout shell, shared
  visual language) keeps League, Tournament, Player Profile, and the
  marketing Website feeling like one product family.

## 7. Security And Audit (Platform-Wide)

- Never expose or persist provider credentials, usable tokens, passwords,
  raw session hashes, unmasked contacts, or raw provider identifiers in
  source, command lines, logs, audits, browser storage/URLs, backups, or
  reports, in any of the five repos.
- Tokens (including the cross-product identity token in Section 2) must be
  random or properly signed, short-lived, purpose-bound, and where
  applicable one-way hashed and single-use. Security-relevant changes revoke
  applicable sessions and tokens.
- Every product logs its own audit trail for authenticated mutations,
  following the shape already established in League's `AuditLogService`:
  authenticated actor, role, target, redacted before/after evidence, reason,
  request id, source, and time. Audit rows are append-only.
- Recovery and delivery flows fail closed across all products, matching
  League's existing rule.
- Production database or application changes, secret generation/storage,
  release tags, and public releases require explicit owner approval in every
  product repo, not just League.

## 8. Approval Boundaries (Platform-Wide)

Before starting a runtime-code or schema phase in any of the five repos,
present the complete proposed:

- implementation and UI/UX behavior;
- database objects, migrations, rollback plan, and version documentation;
- permissions and security effects, including any cross-product
  identity/entitlement impact;
- automated/source/security-negative tests;
- deployment plan, distinguishing autonomous purchase-fulfillment paths
  (Section 5.1) from owner-gated code promotion (Section 5.2).

Receive explicit owner approval before starting that phase. Ask again when a
decision changes code scope, schema, Production, secrets/keys, provider
cost/account ownership, DNS, legal/compliance behavior, real external
delivery, or activation of a live payment provider. Use reversible,
cost-free defaults otherwise.

## 9. Relationship To League's Existing AGENTS.md

`PoolLeagueWeb/AGENTS.md` currently contains both platform-wide rules (which
this document now centralizes) and League-specific rules (score sheet
mechanics, season/history handling, League's specific release-gate
procedure, etc.). This document does not silently replace League's file.

Recommended next step, subject to explicit owner approval before it happens:
split League's current `AGENTS.md` into (a) a short file that states "this
repo inherits `PLATFORM_AGENTS.md`" plus (b) League-specific rules only,
removing the now-duplicated platform-wide sections. This should be proposed
as a reviewable, visible diff against a live production repo — not an
implicit or silent rewrite.

## 10. Open Items Requiring An Explicit Owner Decision

- Exact token format/library (e.g., JWT with a specific signing algorithm)
  and key rotation policy for Account-issued identity tokens.
- Exact shape of the entitlement-check call/claim (embedded in the token vs.
  a live lookup call, and caching policy for the latter).
- Where the shared internal library and `PLATFORM_AGENTS.md` itself should
  live — a sixth `PocketRankingsPlatform` repo is the current proposal.
- Naming/hosting plan for each product's subdomain under `pocketrankings.com`,
  matching the pattern already visible in League's Caddy configuration
  (e.g. `blair8.pocketrankings.com`, `blair8-sb.pocketrankings.com`).
