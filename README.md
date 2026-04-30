# Kriya — engineering notes

The engineering reasoning behind **Kriya** — Insights by Omkar's commercial astronomy / astrology API.

This is a curated public companion to the private Kriya engine. **Not the engine code itself** — just the documents that show *how* it's built and *what* it commits to.

---

## Why publish this

Kriya is a closed-source commercial API (proprietary engine, MIT-licensed SDKs in [Go](https://github.com/Insights-By-Omkar/kriya-go), TypeScript, and Python). But the engineering decisions behind it shouldn't be opaque.

Published here:

- **Accuracy methodology** — what we test against, what tolerances we hold, how the test layers stack
- **Moon-accuracy disclosure** — full transparency on the geometric vs. apparent residual against JPL DE441
- **API versioning policy** — the SemVer contract and stability tiers customers can rely on

If you're evaluating Kriya for production use, this is the engineering you'd ask about in a procurement call. We're publishing it instead.

---

## Contents

### Accuracy

- [`accuracy/accuracy.md`](./accuracy/accuracy.md) — three-layer test strategy: textbook anchors (Meeus worked examples), astronomical-relationship invariants, frozen regression anchors. Per-body tolerances, where they're pinned, what's tested in CI.
- [`accuracy/MOON_CEILING.md`](./accuracy/MOON_CEILING.md) — formal disclosure of the Moon's geometric residual (~21″ vs DE441) and why no analytical lunar theory fitted to DE405 can close the gap. Apparent path is sub-arcsecond and is what every consumer chart hits. Also documents the two paths that *could* push past, and why they're parked.

### API contract

- [`api/API-VERSIONING.md`](./api/API-VERSIONING.md) — versioning policy across all Kriya tracks (astrology, visual, critic). SemVer rules, stability tiers (`experimental` / `beta` / `stable` / `deprecated`), key-scope versioning, deprecation registry.

---

## Related repos

| Repo | Purpose |
|---|---|
| [`kriya-go`](https://github.com/Insights-By-Omkar/kriya-go) | Official Go SDK · MIT · 109+ endpoints · zero runtime dependencies |
| [`insights-astrology-api-case-study`](https://github.com/Insights-By-Omkar/insights-astrology-api-case-study) | High-level architecture, decisions, source policy |
| [`solo-founder-ip-playbook`](https://github.com/Insights-By-Omkar/solo-founder-ip-playbook) | The IP / legal / licensing decisions behind the studio |

The engine code itself stays proprietary — see [kriya.insightsbyomkar.com](https://kriya.insightsbyomkar.com) for product, docs, and pricing.

---

## License

The documents in this repository are MIT-licensed (see [`LICENSE`](./LICENSE)). The Kriya engine and API are proprietary under separate terms; see [studio.insightsbyomkar.com](https://studio.insightsbyomkar.com).

---

*Numbers, not vibes.*
