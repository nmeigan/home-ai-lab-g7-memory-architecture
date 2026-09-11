# Home AI Lab on an 8-Year-Old Laptop

Running 20B–70B parameter LLMs locally on a **Dell G7 (2018, i7-8750H, 32GB RAM, GTX 1050 Ti 4GB VRAM)** — no new GPU, no cloud rental. The lever is memory architecture: a deliberately tuned, simulated unified-memory pool built from physical RAM + NVMe swap, not more hardware.

This repo is the write-up companion to the LinkedIn post and YouTube video ["An 8-Year-Old Laptop, 4GB VRAM, and a 70B Model"](#) — the exact commands and config values referenced there live here, without any personal media (screen recordings/photos stay local).

## Why this exists

Every "run an LLM locally" tutorial assumes either a modern GPU or that you'll settle for a tiny model. This is the opposite case: proving out the actual ceiling of genuinely old, consumer-grade hardware, and treating the exercise as a real infrastructure decision — opportunity cost, sunk cost recovery, token economics, and ESG/carbon footprint — not just a hobby benchmark.

## What's in here

- [`PLAYBOOK.md`](PLAYBOOK.md) — the exact, ordered steps: boot repair, storage layout, WSL2 memory/swap tuning, page file sizing, GPU-offload capping, model selection
- [`ARCHITECTURE.md`](ARCHITECTURE.md) — the memory architecture diagram, the 58GB simulated unified memory pool, and the model-tier decision framework
- [`RESULTS.md`](RESULTS.md) — the real numbers: what model sizes run, at what speed, on this exact hardware
- [`SOVEREIGNTY-AND-ESG.md`](SOVEREIGNTY-AND-ESG.md) — the business case: cost, ESG, and AI sovereignty as one architecture decision, read three ways

## Hardware baseline

| Component | Spec |
|---|---|
| Laptop | Dell G7 7588 (2018) |
| CPU | Intel i7-8750H, 6C/12T, tuned to 3.0GHz |
| RAM | 32GB DDR4 |
| GPU | NVIDIA GTX 1050 Ti, 4GB VRAM (Pascal) |
| Boot drive | Crucial T500 2TB NVMe (PCIe Gen 3) |
| Secondary drive | Crucial MX500 2TB SATA (2.5") |

## The headline result

A 58GB functional memory pool (26GB physical RAM allocated to WSL2 + 32GB NVMe-backed virtual swap) lets this machine:
- Run 14B models at ~20–25 tokens/sec — fully interactive
- Run 32B models at ~10–15 tokens/sec, entirely in physical RAM — the genuine daily-driver sweet spot
- Load 70B-class models (4-bit quantized) without crashing, at ~0.5–1.5 tokens/sec — proof the architecture holds, not a daily driver

See [`RESULTS.md`](RESULTS.md) for the full breakdown and frontier-model baseline comparisons.

## Why this is a board-level question, not just a hobby project

- **Opportunity cost** — hardware already depreciated to zero on the books, converted into productive inference capacity instead of e-waste.
- **Sunk cost recovery** — the "replace it" instinct is the expensive default; memory tuning recovers a capital asset instead of writing it off.
- **Token economics** — every token an LLM generates is billed, directly or as amortized infrastructure cost; bulk/repetitive inference on owned hardware runs at $0 marginal cost per run.
- **ESG** — no new GPU manufactured (manufacturing, not runtime draw, is the dominant share of a GPU's lifecycle carbon footprint), load kept off hyperscale data-center cooling and grid draw.
- **AI sovereignty & responsible AI** — inference runs entirely on hardware you control; no prompt, output, or byte of data routes through a third-party model provider to get an answer.

Full breakdown: [`SOVEREIGNTY-AND-ESG.md`](SOVEREIGNTY-AND-ESG.md).

## License

MIT License — see [`LICENSE`](LICENSE). Documentation and scripts in this repo are shared for reference; no warranty — verify against your own hardware before running destructive disk operations (`diskpart`, partition changes) from `PLAYBOOK.md`.
