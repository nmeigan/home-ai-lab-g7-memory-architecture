# Home AI Lab on an 8-Year-Old Laptop

Running 20B–70B parameter LLMs locally on a **Dell G7 (2018, i7-8750H, 32GB RAM, GTX 1050 Ti 4GB VRAM)** — no new GPU, no cloud rental. The lever is memory architecture: a deliberately tuned, simulated unified-memory pool built from physical RAM + NVMe swap, not more hardware.

This repo is the write-up companion to the LinkedIn post and YouTube video ["An 8-Year-Old Laptop, 4GB VRAM, and a 70B Model"](#) — the exact commands and config values referenced there live here, without any personal media (screen recordings/photos stay local).

## Why this exists

Every "run an LLM locally" tutorial assumes either a modern GPU or that you'll settle for a tiny model. This is the opposite case: proving out the actual ceiling of genuinely old, consumer-grade hardware, and treating the exercise as a real infrastructure decision — opportunity cost, sunk cost recovery, token economics, and ESG/carbon footprint — not just a hobby benchmark.

## What's in here

- [`PLAYBOOK.md`](PLAYBOOK.md) — the exact, ordered steps: boot repair, storage layout, WSL2 memory/swap tuning, page file sizing, model selection
- [`ARCHITECTURE.md`](ARCHITECTURE.md) — the memory architecture diagram and reasoning behind the 58GB simulated unified memory pool
- [`RESULTS.md`](RESULTS.md) — the real numbers: what model sizes run, at what speed, on this exact hardware

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

## License

Documentation and scripts in this repo are shared for reference. No warranty — verify against your own hardware before running destructive disk operations (`diskpart`, partition changes) from `PLAYBOOK.md`.
