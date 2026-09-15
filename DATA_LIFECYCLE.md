# Data Lifecycle And Player Anonymization

Last approved direction: 2026-09-15

## Customer-product cancellation

League and Tournament are separate purchases. When either product's
subscription or entitlement ends, access is disabled immediately. Its isolated
stack and operational data remain recoverable for 61 calendar days, after
which the stack, database, storage, and expired backups are deleted through an
audited workflow.

Normal customer deletion may leave Player Profile with only player-level
results, statistics, exact source identity, and provenance. It must not retain
score sheets, complete brackets, team records, contacts, or customer
operational data.

## Player-data opt-out

An authenticated Account holder may permanently opt out of maintained player
data without deleting the Account or losing purchases and entitlements.

The Account page must explain that the action:

- permanently deletes the public Player Profile, biography, social links,
  media, achievements, statistics, source links, and related profile data;
- prevents future League or Tournament activity from recreating or updating
  the profile;
- replaces the person's participant identity in retained League and
  Tournament history with a unique local anonymous identifier and a neutral
  label;
- preserves historical score sheets, standings, brackets, matches, results,
  placements, and payouts without retaining a link back to the person;
- cannot be reversed, and later opt-in begins a new profile using future
  activity only; and
- leaves Account login, purchases, entitlements, and necessary redacted
  security, transaction, privacy-request, and compliance evidence intact.

## Processing sequence

1. Require an authenticated Account session, current-password confirmation,
   CSRF protection, and an explicit acknowledgment of irreversible effects.
2. Record one idempotent request and one target per in-scope product.
3. Immediately suppress public Player Profile access.
4. Send signed, purpose-bound, replay-safe directives to Player Profile and
   every known local League and Tournament installation.
5. Player Profile deletes the profile graph and retains only a keyed,
   purpose-bound suppression value.
6. Each customer product scrubs identifying participant fields, removes its
   PersonId/profile link, generates a locally unique random surrogate and
   neutral label, and preserves historical foreign-key relationships.
7. Each target returns a signed acknowledgment. Account shows processing or
   failure until every target completes; it never reports a partial request as
   complete.
8. Restored backups replay the deletion ledger before serving traffic.

No product retains a surrogate-to-person mapping. Different products and
customer installations never reuse the same surrogate.
