# API Versioning Policy

This repo hosts multiple commercial API tracks. Each track has its own
SemVer lifecycle and stability guarantees. This document is the
contract.

## Tracks

| Track | URL prefix | Current version | Stability |
|---|---|---|---|
| Astrology | `/api/v1/chart/*`, `/api/v1/reading/*`, `/api/v1/daily/*`, `/api/v1/horoscope/*`, `/api/v1/vedic/*`, `/api/v1/ask-chart` | v1 (engine v7.15.x) | stable |
| Visual Studio | `/api/v1/visual/effects/*`, `/api/v1/visual/character/*` | v1.0.0 (migration from consumer pending) | beta |
| Critic | `/api/v1/critic/*` | v1.0.0 | beta |
| Internal (HMAC-signed) | `/api/internal/*` | not versioned — internal only | n/a |

## Per-track SemVer rules

Each track has its own CHANGELOG at `docs/changelog/<track>-v<major>.md`.

### Patch (x.y.Z)
- Bug fix. Response semantics unchanged. Request semantics unchanged.
- No breaking changes possible.
- Ships in any release.

### Minor (x.Y.z)
- **Additive only.**
- New optional request params allowed (existing calls still work).
- New response fields allowed (existing parsers still work).
- New endpoints under the same version prefix allowed.
- New enum values allowed only for params where unknown-value behavior
  is documented as "fall back to default."
- Never: renaming fields, removing fields, tightening validation,
  changing defaults, reordering arrays with positional meaning.

### Major (X.y.z)
- Breaking changes require a new version prefix: `/api/v2/...`.
- v1 stays alive for **6 months minimum** after v2 ships.
- v1 endpoints emit `Deprecation: true` and `Sunset: <RFC 8594 date>`
  headers once v2 is live.
- Customers on paid plans get written migration guides + a 30-day
  extension on request.

## Stability tiers

Every endpoint carries a `stability` tier in its OpenAPI definition
and in response `meta.stability`:

- **`experimental`** — may change weekly. Don't build production on it.
- **`beta`** — contract is close to stable. Breaking changes possible
  until promoted to `stable`; we'll give 30 days notice on any such
  change via email to customers with active keys scoped to this track.
- **`stable`** — full v1 contract guarantees apply. Any breaking change
  = v2.
- **`deprecated`** — do not integrate new calls. End-of-life date
  published in the response header.

## Key scopes are versioned too

API keys issued via `/api/internal/provision-key` carry versioned scopes:

```json
{ "scopes": ["astrology:v1", "visual:v1", "critic:v1:beta"] }
```

When v2 of any track ships, existing keys stay scoped to v1 until the
customer opts in to v2. v2 invitations go out 60 days before v1 sunset.

## Changelog discipline

- Every PR that touches an API route updates the relevant
  `docs/changelog/<track>-v<major>.md`.
- Every release is tagged in git with the engine SemVer (e.g. `v7.16.0`)
  and the release notes call out per-track impact: "astrology-v1: patch,
  visual-v1: minor (added X effect), critic-v1: no change."

## Deprecation registry

Any endpoint marked `deprecated` must be listed at
`docs/api/deprecations.md` with sunset date, replacement, and
migration notes. Also published on the public docs site.

## This document is normative

Changing this policy requires a commit that updates both the document
and the track-specific changelogs. The headers and scopes in code
follow from here, not the other way around.
