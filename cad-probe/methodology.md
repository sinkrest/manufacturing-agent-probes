# State of Agent CAD — Probe Methodology

**Version:** 1.0
**Last updated:** 2026-04-27
**Author:** Roman Martins
**Project:** [Forkable Factory](https://romanmartins.com/blog/the-forkable-factory)

---

## The Question

**Can an agent take a product spec and modify a parametric CAD model — so another agent can build it — without a human in the loop?**

The "AI + CAD" conversation in 2026 is full of demos: text → 3D model, voice → feature edit, natural language → assembly. Almost none of the conversations grapple with the harder question: once one agent has finished modifying the CAD, can a *different* agent — a sourcing agent, a compliance agent, a manufacturing-routing agent — read the result and act on it?

This probe answers that question concretely, every week, for every major CAD-agent vendor.

---

## The Thesis

If physical products are to be developed the way software is, the CAD layer has to be callable like a function — not just driveable by a human in a GUI, but *agent-usable*: introspectable model state, structured edit operations, machine-readable output, headless execution, programmatic auth, and design-for-manufacturability signals.

The gap between "AI-assisted CAD" and "agent-callable CAD" is the central friction of the design phase of [Forkable Factory](https://romanmartins.com/forkable-factory/map). This probe measures that gap, vendor by vendor, week by week.

---

## How Scoring Works

Each vendor is scored on **8 dimensions** totalling **100 points**. The dimensions map to the three gates of the multi-agent CAD workflow:

| Gate | Dimensions | Weight |
|---|---|---|
| **Edit** (spec → modified geometry) | parametric_edit · feature_tree_introspection · structured_output | 40 pts |
| **Integrate** (CAD agent ↔ other agents) | mcp_compat · headless_mode · auth_accessibility | 35 pts |
| **Output** (geometry → downstream artifacts) | diff_friendly_export · dfm_feedback | 25 pts |

Higher score = closer to being callable by another agent today. Below 50 = not usable in a multi-agent pipeline without significant human bridging.

### The 8 Dimensions

#### 1. parametric_edit — **20 pts** (four-level)

Can the agent modify a feature tree parametrically — not generate from scratch, not regenerate from a prompt, but *edit* existing features?

- **20 pts** — Full parametric assembly editing (multi-part, mate-aware, order-of-operations preserved).
- **10 pts** — Chain of features within one part (multi-step edit on a single body).
- **5 pts** — Single-feature edit only (e.g., resize one dimension, change one extrusion).
- **0 pts** — No edit ability (read-only, image generation, or full-regeneration only).

*Why 20:* This is the foundational gate. An agent that can only generate from scratch can't iterate on a real product spec — every edit is a rewrite. Every other dimension assumes you have something to operate on.

---

#### 2. feature_tree_introspection — **10 pts** (tri-level)

Can the agent read the existing model state — not just see what features exist, but read their parameters, mates, and relationships?

- **10 pts** — Full tree + dimensions (sees current parameter values, named features, and relationships).
- **5 pts** — Names only (lists features but no parameters or values).
- **0 pts** — No introspection (operates blind, can only "do what I tell it" without knowing what's there).

*Why 10:* Without introspection, an agent can't reason about consequences. "Reduce the wall thickness" is a different operation depending on whether wall thickness is currently 2mm or 5mm.

---

#### 3. structured_output — **10 pts** (tri-level)

When the agent finishes an edit, what does it emit? Freeform chat, loose JSON, or schema-compliant structured output that another agent can parse?

- **10 pts** — Strict schema-compliant JSON / structured diff (parseable by next agent without LLM round-trip).
- **5 pts** — Loose JSON (no schema, format varies per request).
- **0 pts** — Freeform text only (chat-style explanation, no machine-readable change-log).

*Why 10:* The whole point of multi-agent pipelines is that one agent's output is another agent's input. Freeform output forces every consumer through an LLM extraction pass. Friction at this gate compounds.

---

#### 4. mcp_compat — **15 pts** (four-level)

Can another agent — Claude Code, an OpenAI agent, an n8n flow, anything that isn't a human — call the CAD agent's edit operations directly?

- **15 pts** — MCP server exposed, or formal tool definitions published (function schema + auth + response shape).
- **10 pts** — REST/HTTP API callable with documented endpoints.
- **5 pts** — Browser-extension lock-in (works inside one host app's UI, no headless / programmatic call path).
- **0 pts** — GUI-only (no external entry point at all).

*Why 15:* This is the integrate gate. A CAD agent that can't be called by another agent isn't a building block — it's a single-player tool with AI inside. The whole multi-agent thesis lives or dies here.

---

#### 5. headless_mode — **10 pts** (tri-level)

Can the agent run without a browser — in CI, in a script, on a server with no display?

- **10 pts** — Pure headless (CLI binary, language SDK, or HTTP service with no display dependency).
- **5 pts** — CLI but requires a display (xvfb-style virtual framebuffer, headless Chromium puppeteering).
- **0 pts** — Browser only.

*Why 10:* If you can't run it on a build server, you can't put it in a pipeline. Headless mode is the difference between "demo" and "infrastructure."

---

#### 6. diff_friendly_export — **10 pts** (tri-level)

When the agent saves the model, what does the file look like under git? Binary opaque, binary with custom diff hooks, or parametric script that diffs line-by-line?

- **10 pts** — Parametric script (OpenSCAD / structured YAML / featurescript / line-level diff).
- **5 pts** — Binary + commit hooks (custom diff tooling, e.g., STEP comparison plugins).
- **0 pts** — Binary only (STEP / STL / opaque proprietary format — diff = "binary differ").

*Why 10:* Software-style version control on hardware is the whole forkable-factory thesis. If you can't see what changed in a commit, you can't review it, branch it, or merge it.

---

#### 7. auth_accessibility — **10 pts** (tri-level)

How hard is it to get from "no account" to "first successful agent call"?

- **10 pts** — OAuth + API key (programmable from first byte; self-serve).
- **5 pts** — Self-serve sign-up (web account, manual key copy, but no human gate).
- **0 pts** — Manual approval gate (sales call, business verification required).

*Why 10:* Reachability. An agent can't complete a sales call. If onboarding requires a human conversation, no agent will ever be the first user.

---

#### 8. dfm_feedback — **15 pts** (tri-level)

Does the agent surface manufacturability concerns — and in a form a downstream sourcing or QA agent can act on?

- **15 pts** — Structured DfM with suggestions (machine-readable warnings + remediation steps the next agent can apply).
- **5 pts** — Basic warnings (extreme cases only — "this hole is too small to drill"). Often human-readable text rather than structured.
- **0 pts** — No DfM signal (agent will happily produce a part that physically can't be made).

*Why 15:* Without DfM feedback, an agent can edit a model that the sourcing agent will then quote and the manufacturing agent will then fail to build. Closing this loop is critical for autonomous operation. Weighted higher than auth (10) because the cost of a bad DfM call is a wasted production run; the cost of a bad auth flow is a delayed call.

---

## Confidence Tiers

Not every dimension can be directly probed every week. Each vendor score carries a confidence tier:

- **live-probed** — Score derived from at least one successful agent call against a sandbox or documented endpoint this run.
- **doc-only** — Score derived from vendor documentation alone (no live endpoint reached this run).

Confidence is a flag, not a multiplier — the score stands, but doc-only scores are subject to higher revision rate. A `doc-only → live-probed` transition often reveals the gap between "what the docs promise" and "what actually works."

For MVP (v1.0), most vendors will be `doc-only`. Adam, Zoo.dev, MecAgent, and Onshape REST are the most likely candidates for `live-probed` upgrades in v2.

---

## Week-over-Week Diff

After each run, scores are compared to the previous week's snapshot:

- **delta_vs_last_week** — integer, positive or negative.
- **notable_changes** — bulleted list of dimension-level changes (e.g., "Added MCP server announcement (doc-confirmed)").

The weekly changelog is published as RSS so the dataset is subscribable.

---

## What Is NOT Measured

By design, the probe does **not** evaluate:

- **Render quality / visual fidelity** — Important for designers; orthogonal to agent-callability.
- **UX polish** — A great human-facing UI doesn't make an agent's job easier.
- **Pricing** — A measure of agent-readiness, not vendor economics.
- **Specific industry vertical fit** — Aerospace vs consumer vs medical CAD have different needs; v1 measures generic agent-readiness.
- **Generative quality** — Probably the loudest current marketing axis. We measure edit-callability, which is a stricter bar.

v2 may introduce a separate dimension family for these. For v1.0, we keep the scope to agent-readiness of the CAD edit / integrate / output surface only.

---

## What The Probe Does (and Doesn't Do)

### Does

- Fetch public documentation pages via WebFetch.
- Read published API references, plugin marketplaces, and SDK repos where available.
- Score each vendor per the rubric above.
- Compute week-over-week diffs against the previous snapshot.
- Publish results to [romanmartins.com/cad-agents](https://romanmartins.com/cad-agents) with RSS changelog.

### Does NOT

- Place orders against any vendor's API.
- Transmit credentials in published outputs.
- Create accounts beyond what a public sandbox allows.
- Scrape pages the vendor's `robots.txt` disallows.
- Publish any vendor's private keys.

All probes are read-only against documented surfaces. Scoring is transparent, auditable, and reproducible — every score has a `source_url`.

---

## Onshape REST as a Baseline

Onshape REST is included as a vendor row deliberately. It is the substrate Adam (and several future entrants) ride on top of. Probing the substrate alongside the agents that use it surfaces:

- **A control.** The delta between Adam's score and Onshape REST's score is what Adam actually adds, dimension by dimension.
- **Honest credit.** Some of Adam's `feature_tree_introspection` is Onshape's underlying API; some is Adam's interpretation layer. The leaderboard surfaces both.
- **A precedent.** Future SolidWorks AI / Creo AI entrants will want their substrates (SolidWorks REST / Creo Web Toolkit) similarly baseline-probed.

The leaderboard explicitly notes Onshape REST as substrate, not direct competitor.

---

## Example: Adam v1.0 (illustrative)

Based on public information at `adam.new`, Y Combinator profile, TechCrunch coverage (Oct 2025), and Onshape App Store listing:

| Dimension | Score | Notes |
|---|---|---|
| 1. parametric_edit | 20 | Full parametric assembly editing demonstrated in launch demos; multi-part Onshape models |
| 2. feature_tree_introspection | 10 | Reads existing feature tree (must, to apply edits) |
| 3. structured_output | 5 | Diff applied to Onshape model; no public schema for the diff format itself |
| 4. mcp_compat | 5 | Browser-extension lock-in; no public REST or MCP path documented |
| 5. headless_mode | 0 | Chrome extension required |
| 6. diff_friendly_export | 0 | Outputs Onshape internal format; binary STEP export downstream |
| 7. auth_accessibility | 5 | Self-serve sign-up + Onshape App Store install |
| 8. dfm_feedback | 5 | Some DfM signal in demos (e.g., wall-thickness feedback); no structured DfM output |
| **Total** | **50** | Confidence: doc-only |

*(Illustrative example. Actual scores reproduced weekly by `/cad-probe` against live documentation.)*

---

## Methodology Changelog

### v1.0.1 — 2026-04-30
- Added MecAgent (SolidWorks/Inventor desktop plugin) as 8th MVP vendor for substrate-coverage parity with Adam (Onshape).

### v1.0 — 2026-04-27
- Initial 8-dimension rubric, 100 points total.
- 7 MVP vendors (adam, zoo, spline, onshape_rest, fusion_ai, shapr3d, openscad_llm).
- Doc-only + live-probed confidence tiers.
- Onshape REST included as baseline substrate (not direct competitor).

---

## Audit This

If you're a vendor who disagrees with your score: open an issue at [github.com/forkable-factory](https://github.com/forkable-factory) or email [roman@romanmartins.com](mailto:roman@romanmartins.com). Every score has a `source_url` field — if the URL is wrong or stale, we'll re-probe.

If you're a builder who sees a dimension we're missing: same channel. Methodology evolves openly.

---

*This probe is part of the [Forkable Factory](https://romanmartins.com/blog/the-forkable-factory) research project — measuring the gap between "physical products" and "software-style development."*
