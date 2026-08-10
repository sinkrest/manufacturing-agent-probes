# State of Manufacturing APIs — Probe Methodology

**Version:** 1.0
**Last updated:** 2026-04-16
**Author:** Roman Martins
**Project:** [Forkable Factory](https://romanmartins.com/blog/the-forkable-factory)

---

## The Question

**Can an agent go from `product spec` → `quote` → `order` → `delivered parts` without a human in the loop?**

This is the question. Everyone in the "AI + manufacturing" conversation dances around it. This probe answers it concretely, every week, for every major distributed manufacturing vendor.

---

## The Thesis

If physical products are to be developed the way software is, the supply chain has to be callable like an API — not just *have* an API, but be *agent-usable*: self-serve auth, structured responses, programmatic ordering, queryable status, and rapid feedback loops.

The gap between "API exists" and "agent can use it" is the central friction of Forkable Factory. This probe measures that gap, vendor by vendor, week by week.

---

## How Scoring Works

Each vendor is scored on **8 dimensions** totalling **100 points**. The dimensions map to the four gates of the agent workflow:

| Gate | Dimensions | Weight |
|---|---|---|
| **Quote** (spec → price) | Instant quote from CAD · Structured JSON pricing · DFM feedback · Time-to-first-quote | 50 pts |
| **Order** (price → transaction) | API-only order placement · Auth accessibility | 30 pts |
| **Status** (order → delivery visibility) | Order status queryable · Rate limits documented | 20 pts |

Higher score = closer to being callable by an agent today. Below 50 = not usable by an agent without significant human bridging.

### The 8 Dimensions

#### 1. Instant quote from CAD upload — **20 pts** (binary)

Can the vendor return a quote for a STEP or STL file uploaded via API call, with no human intervention?

- **20 pts** — Yes, documented endpoint accepts a CAD file and returns a quote.
- **0 pts** — No automated quoting, or requires human review before pricing returned.

*Why 20:* This is the foundational gate. Without programmatic quoting, the rest is moot.

---

#### 2. Structured JSON pricing response — **15 pts** (binary)

Is the pricing returned as parseable structured data (JSON, XML), not PDF, image, or email?

- **15 pts** — Structured response an agent can parse directly.
- **0 pts** — Human-readable only (PDF, email, quote document).

*Why 15:* Unstructured output forces an LLM round-trip to extract numbers. Acceptable but friction.

---

#### 3. API-only order placement — **20 pts** (tri-level)

Can an order be placed end-to-end via API, or does a human have to click somewhere?

- **20 pts** — Full order placement via API, including payment.
- **10 pts** — Partial: API can create a draft order but requires human confirmation or payment step.
- **0 pts** — No API ordering; sales-call-required.

*Why 20:* The transaction gate. Even partial credit is significant progress.

---

#### 4. Order status queryable — **15 pts** (tri-level)

After ordering, can an agent know where the order is without a human?

- **15 pts** — Webhook notifications on status change.
- **10 pts** — Polling endpoint returns current status.
- **0 pts** — Email-only or no status API.

*Why 15:* Without status, an agent has no idea when its parts arrive. Webhooks > polling because they're push-based.

---

#### 5. Auth accessibility — **10 pts** (tri-level)

How hard is it to get from "no account" to "first successful API call"?

- **10 pts** — Self-serve auth (OAuth2, sign-up-and-generate-key).
- **5 pts** — API key on request (fill a form, wait).
- **0 pts** — Sales call required.

*Why 10:* This is about reachability. An agent can't complete a sales call.

---

#### 6. Rate limits documented — **5 pts** (binary)

Is there clear documentation on request quotas and rate limits?

- **5 pts** — Documented.
- **0 pts** — Not documented or hidden.

*Why 5:* Less critical but signals API maturity. Undocumented limits create silent failure modes for agents.

---

#### 7. DFM feedback in API response — **10 pts** (binary)

Does the quote response include design-for-manufacturing warnings (wall thickness, tolerances, unprintable features)?

- **10 pts** — Yes, structured DFM data.
- **0 pts** — Quote returned with no manufacturability signal.

*Why 10:* Without DFM feedback, an agent can order a part that physically can't be made. Closing this loop is critical for autonomous operation.

---

#### 8. Time-to-first-quote speed — **5 pts** (tri-level)

How fast does the quote come back?

- **5 pts** — Under 1 minute (sync response).
- **3 pts** — 1 minute to 1 hour (async but fast).
- **0 pts** — Over 1 hour (human-review gated).

*Why 5:* Speed matters for iteration loops, but slow automated quoting still beats human-only. Low weight because the other gates dominate.

---

## Confidence Tiers

Not every dimension can be directly probed every week. Each vendor score carries a confidence tier:

- **live-probed** — Score derived from at least one successful API call against a sandbox or documented endpoint this run.
- **doc-only** — Score derived from vendor documentation alone (no live endpoint reached this run).

Confidence is a flag, not a multiplier — the score stands, but doc-only scores are subject to higher revision rate. A `doc-only → live-probed` transition often reveals the gap between "what the docs promise" and "what actually works."

---

## Week-over-Week Diff

After each run, scores are compared to the previous week's snapshot:

- **delta_vs_last_week** — integer, positive or negative.
- **notable_changes** — bulleted list of dimension-level changes (e.g., "Added webhook endpoint for order status (doc-confirmed)").

The weekly changelog is published as RSS so the dataset is subscribable.

---

## What Is NOT Measured

By design, the probe does **not** evaluate:

- **Price competitiveness** — This is a measure of agent-readiness, not vendor economics.
- **Part quality** — Outside the scope of API accessibility.
- **Lead time** — Some signal via TTFQ, but full fulfillment time is a vendor-ops question.
- **Geographic coverage** — Important for real builds, not for API-surface assessment.
- **Payment terms / net terms** — Enterprise-specific, not relevant to MVP scoring.

v2 may introduce a separate dimension family for these. For v1.0, we keep the scope to agent-readiness of the API surface only.

---

## What The Probe Does (and Doesn't Do)

### Does

- Fetch public documentation pages via WebFetch.
- Read published OpenAPI / Swagger specs where available.
- Score each vendor per the rubric above.
- Compute week-over-week diffs against the previous snapshot.
- Publish results to [romanmartins.com/manufacturing-apis](https://romanmartins.com/manufacturing-apis) with RSS changelog.

### Does NOT

- Place real orders.
- Transmit payment information.
- Create accounts beyond what a public sandbox allows.
- Scrape pages the vendor's `robots.txt` disallows.
- Publish any vendor's private keys or credentials.

All probes are read-only against documented API endpoints. The probe's scoring is transparent, auditable, and reproducible.

---

## Example: Xometry v1.0 (illustrative)

Based on public documentation at `xometry.com/api/docs`:

| Dimension | Score | Notes |
|---|---|---|
| 1. Instant quote from CAD | 20 | Documented `/v1/quotes` endpoint accepts STEP/STL upload |
| 2. Structured JSON pricing | 15 | Response is JSON with `price`, `currency`, `lead_time_days` |
| 3. API-only order placement | 10 | Order API documented but final checkout requires human confirmation in docs |
| 4. Order status queryable | 10 | Polling endpoint documented; no webhook in public docs |
| 5. Auth accessibility | 5 | API key available on request (form submission) |
| 6. Rate limits documented | 0 | Not found in public docs |
| 7. DFM feedback | 10 | Quote response includes `manufacturability_issues` array |
| 8. Time-to-first-quote | 2 | Async response, quote delivered within minutes per docs |
| **Total** | **72** | Confidence: doc-only |

*(Illustrative example. Actual scores reproduced weekly by `/ff-api-probe` against live documentation.)*

---

## Methodology Changelog

### v1.0 — 2026-04-16
- Initial 8-dimension rubric, 100 points total.
- 7 MVP vendors.
- Doc-only + live-probed confidence tiers.

---

## Audit This

If you're a vendor who disagrees with your score: open an issue at [github.com/forkable-factory](https://github.com/forkable-factory) or email [roman@romanmartins.com](mailto:roman@romanmartins.com). Every score has a `source_url` field — if the URL is wrong or stale, we'll re-probe.

If you're a builder who sees a dimension we're missing: same channel. Methodology evolves openly.

---

*This probe is part of the [Forkable Factory](https://romanmartins.com/blog/the-forkable-factory) research project — measuring the gap between "physical products" and "software-style development."*
