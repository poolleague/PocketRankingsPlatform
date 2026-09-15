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

Status: APPROVED by the owner on 2026-09-13. Every product repository may
formally inherit this file. A product's local rules may add stricter or
product-specific requirements but may not silently weaken these shared rules.

## 0. Repository Coordination And Evidence

- Only one agent is active in a repository at a time. Work in another product
  repository is not permission to modify, switch, or clean an active agent's
  workspace.
- Always work on a feature branch, never directly on `main`. At the start of
  each session inspect and report the branch, HEAD, upstream, working tree,
  and active worktrees before taking another action.
- Preserve unrelated or dirty work. If ownership or overlap is unclear, stop
  and ask the owner.
- Base conclusions on verified evidence. Check changing facts rather than
  carrying them forward, and state genuine uncertainty instead of filling it
  with a plausible explanation.
- A working agreement or process change is permanent across later tasks and
  applicable product repositories unless the owner explicitly limits it.
- Before proposing or starting issue, release, or product work, reconcile the
  request against live relevant issues and inspect the complete body,
  comments, relationships, milestone, and linked pull requests of plausible
  matches. Do not mutate issue state without authorization.

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
- Shared, non-product-specific contracts — session/token verification, audit
  logging, the outbox/inbox event envelope, and entitlement checks — are
  versioned centrally. Until a second real consumer establishes the reusable
  boundary, each product may implement the smallest local version against the
  shared contract. Once two consumers exist, move proven common code into an
  internal versioned library rather than allowing copies to drift. Creating or
  changing that shared runtime library is a separately reviewed phase because
  a defect there affects every product.
- A shared, lightweight design system (CSS variables, layout shell, shared
  visual language) keeps League, Tournament, Player Profile, and the
  marketing Website feeling like one product family.

### 6.1 Inline Code And Documentation

- Every new non-trivial class, view-model, and private/public method gets a
  short header comment explaining why it exists or the non-obvious constraint
  it preserves. Comment reasoning, boundaries, history, and deliberate quirks;
  do not restate visible code.
- Each product maintains current-shape, important-path, database, testing,
  deployment, and release-handoff documentation appropriate to its maturity.
  When a file, method, schema object, test inventory, route, or deployment fact
  changes, search the maintained documentation and update current/future
  descriptions in the same change. Preserve dated historical narratives.
- Platform-rule changes require a same-task review of every product's local
  `AGENTS.md` and `CLAUDE.md`. Each product used by both agents keeps those
  local files synchronized below their intentionally different introductions.

### 6.2 Shared Product Experience

- The products share brand tokens, component semantics, status language,
  account patterns, responsive behavior, and accessibility expectations so
  customers experience one Pocket Rankings family.
- Product workflows and navigation remain fit for their users; visual
  consistency does not mean forcing League scoring, Tournament operations,
  Player Profile, Account, and marketing into one navigation model.
- Keep navigation shallow and task-oriented. Reuse established actions,
  statuses, confirmations, and owner-facing terms when behavior is equivalent.
  New distinctions must be explicit and consistent across UI, Help, tests,
  audits, and documentation.
- Shared branding may not reduce contrast, focus visibility, touch targets,
  keyboard operation, reduced-motion behavior, or assistive semantics.

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

### 7.1 External Tools And Resource Discipline

- Use connected design, document, storage, messaging, browser, and research
  tools only when they materially benefit the current task. Prefer local
  repository evidence, then purpose-built connectors, then browser control.
- Product content may enter only a private, owner-controlled external
  destination whose sharing, ownership, training, retention, and recipient
  state has been verified. Never publish to a public gallery or marketplace,
  change sharing, invite collaborators, accept legal terms, or transfer
  ownership without explicit approval.
- Send the minimum necessary content. Never send secrets, credentials,
  Production or personal data, security internals, private deployment details,
  or recovery material to an external tool.
- Treat usage, builds, browser work, remote operations, and validations as
  limited resources. Batch independent read-only checks, reuse current
  evidence, run focused checks before full gates, and never conserve usage by
  weakening security or required coverage.

## 8. Approval Boundaries (Platform-Wide)

Before starting a runtime-code or schema phase in any of the five repos,
present the complete proposed:

- implementation and UI/UX behavior;
- database objects, migrations, rollback plan, and version documentation;
- permissions and security effects, including any cross-product
  identity/entitlement impact;
- automated/source/security-negative tests, plus responsive, accessibility,
  performance/load, migration, recovery, and release-gate coverage appropriate
  to the affected behavior;
- deployment plan, distinguishing autonomous purchase-fulfillment paths
  (Section 5.1) from owner-gated code promotion (Section 5.2).

Receive explicit owner approval before starting that phase. Ask again when a
decision changes code scope, schema, Production, secrets/keys, provider
cost/account ownership, DNS, legal/compliance behavior, real external
delivery, or activation of a live payment provider. Use reversible,
cost-free defaults otherwise.

### 8.1 Database, Validation, And Release Discipline

- PostgreSQL is the Production database. Every schema object and later field,
  index, key, constraint, or ownership change is documented with its
  introduction/change version.
- Migrations are idempotent, preserve current data, verify integrity, fail
  safely, and have a tested rollback or forward-recovery plan. A Sandbox pass
  never authorizes a Production schema change.
- Recalculate test and gate inventories from the exact candidate. Run the
  smallest affected checks while implementation is changing, then freeze one
  intentional candidate before release-level validation. A changed runtime
  candidate needs fresh applicable evidence.
- Long-running remote operations use an operation-specific bounded monitor and
  their process/report/artifact evidence as the completion signal. Do not
  restart a timeout until that evidence has been inspected.
- A Production candidate requires a Release build, intentional commit and
  push, exact Sandbox validation, protected rollback evidence, and explicit
  owner approval. Deploy only the Git tree that passed. Production tags and
  public releases require separate approval and immutable tags are never moved
  or reused.
- Preserve a named rollback image and protected backup before Production data
  or application changes. Verify health, version, schema, isolation,
  containers, endpoints, and logs after promotion without inventing or
  resetting credentials for unavailable authenticated checks.

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

- Account currently issues a locally implemented five-minute RS256 JWT and
  publishes JWKS. Before another product consumes it, approve the durable
  token contract, issuer/audience rules, key storage, rollover, cache, outage,
  and revocation behavior; the existing implementation is evidence, not an
  implicit platform-wide security decision.
- Exact shape of the entitlement-check call/claim (embedded in the token vs.
  a live lookup call, and caching policy for the latter).
- `PLATFORM_AGENTS.md` lives in this approved `PocketRankingsPlatform`
  repository. The package layout and release/version process for the future
  shared runtime library remain open until a second consumer exists.
- Naming/hosting plan for each product's subdomain under `pocketrankings.com`,
  matching the pattern already visible in League's Caddy configuration
  (e.g. `blair8.pocketrankings.com`, `blair8-sb.pocketrankings.com`).

## 11. SOC 2 Readiness And Control Evidence

The owner selected SOC 2 Type I readiness for Security, Availability, and
Confidentiality first, followed by Type II after controls have operated for an
auditor-selected observation period. Privacy and Processing Integrity remain
design requirements where applicable even when they are outside the initial
report scope.

- Never claim that Pocket Rankings is SOC 2 compliant, certified, or audited
  unless an independent licensed CPA firm has issued the applicable report and
  the claim accurately reflects its scope and period. Repository controls are
  readiness measures, not an attestation.
- Maintain a versioned system description, product/data-flow inventory, asset
  inventory, data classification, control matrix, named control owners, risk
  register, vendor register, and evidence index. Review them at least annually
  and after material architecture, provider, security, or data-lifecycle
  changes.
- Production and administrative access requires named individual identities,
  phishing-resistant MFA where supported, least privilege, separation of
  duties for high-risk changes, prompt offboarding, and periodic documented
  access reviews. Shared credentials and unrecorded emergency access are
  prohibited.
- Every change uses a traceable request, feature branch, reviewable diff,
  applicable automated validation, approval, immutable commit, deployment
  evidence, and rollback plan. Emergency changes require recorded authority,
  bounded scope, validation, and timely retrospective review; urgency never
  permits silent or untracked changes.
- Maintain dependency, secret, static-analysis, configuration, and
  vulnerability scanning. Findings have a recorded severity, owner, due date,
  remediation or formally approved time-bounded exception, and verification.
  Never weaken a scan or threshold to obtain a passing result.
- Encrypt confidential data in transit and at rest. Secrets use an approved
  secret store, narrow service identities, rotation, revocation, inventory,
  and access evidence. Secrets never enter source, logs, screenshots, test
  artifacts, tickets, or chat.
- Security, authentication, authorization, privacy, administrative, backup,
  restore, deployment, configuration, and integration events produce bounded,
  tamper-resistant, time-synchronized evidence. Alerting is tested; sensitive
  content is minimized and redacted.
- Maintain approved incident-response, escalation, evidence-preservation,
  customer-communication, business-continuity, disaster-recovery, backup,
  and restoration procedures. Test them on a documented cadence and track
  corrective actions to closure. Production RTO, RPO, availability targets,
  and alert ownership require explicit owner approval before launch.
- Assess service providers before use and periodically thereafter for data
  access, security, availability, confidentiality, privacy, retention,
  incident notification, subcontractors, and exit capability. Provider terms
  or attestations do not replace Pocket Rankings' own controls.
- Security and privacy training, confidentiality commitments, background
  screening decisions, personnel onboarding/offboarding, and policy
  acknowledgments are company controls outside source code but must have named
  owners and retained evidence before an audit period begins.
- Evidence retention, privacy deletion, backup expiration, and legal holds
  follow the approved data-lifecycle policy. Compliance evidence must prove
  that an action occurred without retaining deleted personal or secret data.
- Record every control exception with scope, risk, compensating control,
  approver, expiration, and closure evidence. Agents may surface or document
  gaps but may not self-approve exceptions or represent readiness as achieved.

See `SOC2_READINESS.md`, `SOC2_CONTROL_MATRIX.md`, `RISK_REGISTER.md`,
`EVIDENCE_INDEX.md`, `ASSET_VENDOR_REGISTER.md`,
`INCIDENT_RESPONSE_PLAN.md`, `BUSINESS_CONTINUITY_PLAN.md`, and
`CONTROL_EXCEPTION_REGISTER.md` for the readiness records. Templates and
planned controls are not operating evidence.

## 12. Purchase Provisioning And Data Lifecycle

- League and Tournament are separate products and separate purchases. Buying
  one never silently provisions or grants the other.
- Account, Player Profile, and the marketing Website are shared services.
  Each purchased League or Tournament customer product receives an isolated
  runtime stack, database, storage, configuration, credentials, backups, and
  hostname from the same immutable approved product image.
- Purchase confirmation, entitlement grant, provisioning, readiness, and
  customer notification are distinct idempotent states. A payment webhook
  never runs infrastructure work inline and never marks a product ready before
  health, schema, isolation, and administrator-handoff checks pass.
- Cancellation disables product access immediately. Operational League or
  Tournament data remains recoverable for 61 calendar days beginning when
  that product's subscription or entitlement ends, then the product stack,
  database, storage, and expired backups are deleted through an evidenced,
  retry-safe workflow. A specifically authorized legal hold pauses deletion
  without restoring customer access.
- Normal customer-product deletion may leave Player Profile with only bounded
  player-level results, statistics, exact source identity, and provenance.
  Score sheets, brackets, teams, contacts, and customer operational data do
  not transfer to or remain in Player Profile.
- An authenticated player's global player-data opt-out is different from
  customer-product cancellation. It keeps Account login, purchases,
  entitlements, and necessary security/transaction records active while
  immediately and irreversibly erasing Player Profile data and anonymizing
  the person's participant identity inside retained League and Tournament
  history.
- Anonymization replaces each local participant with a product-installation-
  specific random surrogate and neutral label such as `Deleted player A7K4`.
  Remove names, contacts, PersonId, profile/source links, media, and other
  identifying fields. Never retain a mapping back to the person or reuse one
  surrogate across products or customer installations.
- Historical score sheets, standings, brackets, matches, results, placements,
  and payouts may continue referencing the anonymous local participant. A
  privacy-safe audit proves anonymization occurred without retaining erased
  values.
- Player Profile keeps only a keyed, purpose-bound suppression value needed to
  reject accidental recreation or future projections. Opting back in creates
  a new profile from future activity; erased data is never silently rebuilt.
- A destructive privacy request requires recent authentication, explicit
  irreversible-action confirmation, CSRF protection, idempotency, signed
  product directives, per-target acknowledgments, and visible completion or
  failure state. Do not claim completion until every in-scope product has
  acknowledged its action.
- Backups containing erased data expire under policy and remain inaccessible
  for ordinary use. Any authorized restoration must replay the durable
  deletion ledger before the restored system can serve traffic.

See `DATA_LIFECYCLE.md` and `AUTOMATED_PROVISIONING.md` for the approved
cross-product workflows.
