# Cost, ESG, and AI Sovereignty — One Architecture Decision, Read Three Ways

This document is the business case underneath `PLAYBOOK.md` and `ARCHITECTURE.md` — why "run it locally on hardware you already own" is a board-level decision, not a hobbyist one.

## The same lever, three reporting lines

| Lens | What the memory-tuning decision produces |
|---|---|
| **Finance (CFO)** | Zero marginal cost per inference run on hardware already depreciated to zero book value — a capital asset recovered instead of written off as e-waste. Opportunity cost turned into realized capacity; the "replace it" instinct is the expensive default, reversed. |
| **Sustainability (ESG/CDO)** | No new GPU manufactured — manufacturing, not runtime power draw, is the dominant share of a GPU's lifecycle carbon footprint. Bulk/repetitive inference kept off hyperscale data-center cooling and grid draw. A depreciated laptop put back to real work instead of landfill is a genuine circular-economy outcome, not an offset claim. |
| **Governance (CAIO/CTO, Responsible AI)** | Inference runs entirely on hardware you control. No prompt, no output, and no byte of data is routed through a third-party model provider to get an answer — no external processor to disclose in a data-processing record, no dependency on an outside vendor's uptime, pricing, or terms of service. |

Cost efficiency, ESG performance, and AI sovereignty are not three competing priorities here. They are one architecture decision — tier the memory, cap GPU offload to measured usage, keep model storage off cloud sync — read from three different organizational vantage points.

## Why sovereignty is the quieter twin

Cost and ESG get most of the airtime in local-inference discussions. Sovereignty is usually treated as a separate, harder problem — solved by expensive on-prem GPU clusters or a private-cloud contract. This project is the counter-proof at the smallest possible scale: a single depreciated laptop, tuned correctly, already satisfies the core sovereignty requirement — the inference boundary never leaves hardware you control — for the class of workload it can actually carry (see `RESULTS.md` for the real tier-by-tier ceiling).

That doesn't make it a substitute for enterprise-scale sovereign infrastructure. It makes it the smallest legitimate proof point that the pattern works before scaling the same discipline to a real fleet.

## Build vs. buy — where this pattern fits on a real portfolio

- **Cloud inference** — opex, fastest start, vendor lock-in, per-token billing, and every request leaves your infrastructure boundary by default.
- **Local/edge inference (this pattern)** — capex-free if hardware is already owned, a depreciating asset either way, requires the memory-tuning discipline in `PLAYBOOK.md`, keeps the inference boundary inside infrastructure you control, best suited to bulk/repetitive/non-time-critical workloads.
- **Engine-native inference** — inference runs inside the transactional/data engine itself (a separate architecture pattern, not covered in this repo).

For a CDO/CAIO/CTO/CXO building the case for in-house AI infrastructure: this is the fastest legitimate proof that "sweat existing hardware first, under your own governance boundary" is a real option before the next capex or vendor-contract cycle — scored on the same cost / ESG / sovereignty basis as any cloud-vs-on-prem decision.

## Reproducibility note

This is a single-machine proof point, not a fleet-scale benchmark or a compliance certification. Absolute token/sec numbers are specific to the exact hardware in `RESULTS.md` and will vary elsewhere. The reusable part is the *pattern* — tier the memory deliberately, measure before capping GPU offload, keep model storage off cloud sync, and treat "where does inference actually run" as an explicit governance decision — not the specific numbers.
