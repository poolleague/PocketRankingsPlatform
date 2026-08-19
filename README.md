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

Draft. `PLATFORM_AGENTS.md` is pending final owner review — see its
"Status" line and Section 10 (Open Items) before treating it as final.
