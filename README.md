# Manufacturing Agent Probes

**Can a software agent actually get a physical part made?**

Everyone in the "AI + manufacturing" conversation dances around that question. These two probes answer it concretely, vendor by vendor, with every score linked to the documentation page it came from.

This repository is the **open dataset and methodology**. Two probes:

| Probe | Question | Vendors |
|---|---|---|
| **[api-probe](api-probe/)** | Can an agent go spec → quote → order → delivered parts with no human in the loop? | Xometry, Protolabs, Hubs, PCBWay, JLCPCB, Shapeways, Fictiv |
| **[cad-probe](cad-probe/)** | Can an agent produce and modify real parametric CAD? | Zoo.dev, Onshape, Adam, MecAgent, Autodesk Fusion AI, Shapr3D AI, Spline AI, OpenSCAD+LLM |

Each vendor is scored on **8 dimensions totalling 100 points**. Higher = closer to being callable by an agent today. Below 50 = not usable without significant human bridging.

---

## What the data currently says

**Manufacturing APIs** (run 2026-06-30) — the striking result is how *low* the field sits. Four of seven vendors score at or near zero: the marketing says "API", the documentation describes a sales conversation.

| Score | Vendor |
|---:|---|
| 88 | Shapeways |
| 65 | PCBWay |
| 38 | JLCPCB |
| 5 | Xometry |
| 0 | Hubs (Protolabs Network) |
| 0 | Protolabs |
| 0 | Fictiv |

**Agent CAD** (run 2026-06-15) — the substrate is further along than the agents built on top of it. Onshape's plain REST API (65) outscores every AI-branded CAD product except Zoo.dev.

| Score | Vendor |
|---:|---|
| 90 | Zoo.dev (KittyCAD) |
| 65 | Onshape REST API *(substrate baseline, not a competitor)* |
| 50 | OpenSCAD + LLM plugins |
| 50 | Autodesk Fusion 360 AI |
| 35 | MecAgent |
| 35 | Adam |
| 10 | Spline AI |
| 5 | Shapr3D AI |

---

## Coverage — read this before citing

This is a **snapshot dataset with gaps**, not a live feed. Be precise about it:

- **api-probe:** 8 runs, **2026-04-16 → 2026-06-30**
- **cad-probe:** 4 runs, **2026-04-30 → 2026-06-15**

The probe is designed to run weekly; in practice it ran roughly fortnightly and has been paused since the dates above. Scores were flat across the final several api-probe runs — the leaderboard did not move between 2026-05-11 and 2026-06-30 — so the picture is likely still broadly accurate, but **treat every score as "as of its run date"** and re-verify the `source_url` before relying on it for a decision.

If a vendor has shipped something since, the score is stale by construction. Open an issue.

## Layout

```
api-probe/
  methodology.md        the 8-dimension rubric, v1.0 — how every point is earned
  vendors.yaml          the registry: what gets probed and where
  history.jsonl         append-only time-series, one line per vendor per run
  snapshots/*.json      full run output, including probe notes and headlines
cad-probe/              same shape
```

`history.jsonl` is the file you want for charting movement over time. `snapshots/` carries the prose: what changed that week and why the score did or didn't move.

## How scoring works

Full rubrics in [`api-probe/methodology.md`](api-probe/methodology.md) and [`cad-probe/methodology.md`](cad-probe/methodology.md). The short version for the API probe — dimensions map to the four gates of an agent workflow:

| Gate | Weight |
|---|---|
| **Quote** — instant quote from CAD, structured JSON pricing, DFM feedback, time-to-first-quote | 50 pts |
| **Order** — API-only order placement, auth accessibility | 30 pts |
| **Status** — order status queryable, rate limits documented | 20 pts |

Two properties worth knowing:

- **It is a documentation probe.** Scores reflect what a developer can discover and use from public documentation without a sales call. That is deliberate — "you can have an API if you talk to our team" is precisely the friction being measured — but it means a vendor with a great private API scores badly, and that is the intended reading, not a bug.
- **Every score carries a `source_url`.** If a score is wrong, the URL is checkable. That is the whole basis on which corrections are accepted.

## Corrections

**Vendors:** if you disagree with your score, open an issue. Every dimension has a `source_url`; if it's wrong or stale, point at the right one and the vendor gets re-probed. No score changes without a documentation link.

**Everyone else:** issues and PRs welcome, particularly for vendors missing from the registry.

## License

- **Data** (`history.jsonl`, `snapshots/`, `vendors.yaml`) — [CC BY 4.0](LICENSE). Use it, chart it, publish from it; credit the Forkable Factory probe and link back.
- **Methodology documents** — same terms.

Attribution string: *Forkable Factory Manufacturing Agent Probes, Roman Martins — github.com/forkable-factory/manufacturing-agent-probes*

---

Part of [Forkable Factory](https://romanmartins.com/blog/the-forkable-factory) — measuring the gap between "physical products" and software-style development. Leaderboards render at [romanmartins.com/manufacturing-apis](https://romanmartins.com/manufacturing-apis) and [romanmartins.com/cad-agents](https://romanmartins.com/cad-agents).
