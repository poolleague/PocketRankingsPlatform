# Pocket Rankings Platform

Shared governance and architecture documentation for the Pocket Rankings
product family: `PoolLeagueWeb`, `PocketRankingsAccount`,
`PocketRankingsWeb`, `PocketRankingsTournament`, and
`PocketRankingsPlayerProfile`.

This repo is intentionally **docs-only**. It holds the rules and
conventions every product repo inherits — not shared runtime code. See
`PLATFORM_AGENTS.md` for the full set of shared rules.

Shared runtime code (session/token verification, audit logging, event
envelope) is deliberately deferred until a second product actually needs
it, so the interface is designed against real usage rather than a guess.
Until then, each product implements its own local version of that logic,
matching the pattern already established in `PoolLeagueWeb`.

## Status

`PLATFORM_AGENTS.md` was approved by the owner on 2026-09-13. Its explicitly
listed open security and hosting decisions remain owner-gated; approval of the
working rules does not silently decide those product-design questions.
