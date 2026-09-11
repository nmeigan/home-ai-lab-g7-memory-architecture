# Architecture — Simulated Unified Memory on Consumer x86 Hardware

## The core idea

Apple markets "unified memory" as a headline feature of its own silicon — CPU, GPU, and memory sharing one addressable pool so a model isn't bottlenecked by a small, separate VRAM tier. Consumer x86 laptops don't have that hardware-level design. But you can approximate the *effect* — a single large addressable pool that a model's runtime can spill across — by deliberately tiering physical RAM and fast NVMe swap instead of leaving each layer to its OS defaults.

```
┌─────────────────────────────────────────────────────────────┐
│                  58GB Functional Compute Pool                │
│                                                                │
│  ┌──────────────────────────┐   ┌──────────────────────────┐ │
│  │  Physical RAM (WSL2)      │   │  NVMe Virtual Swap        │ │
│  │  26 GB                    │ + │  32 GB                    │ │
│  │  ~25,000+ MB/s            │   │  ~3,500 MB/s (PCIe Gen 3) │ │
│  └──────────────────────────┘   └──────────────────────────┘ │
│                                                                │
│  4GB GTX 1050 Ti VRAM: first few model layers only            │
└─────────────────────────────────────────────────────────────┘
```

## Why this isn't "just add more swap"

Swap is not extra RAM — it's an emergency overflow lane, roughly 7x slower than physical RAM on this hardware (NVMe ~3,500 MB/s vs. DDR4 ~25,000+ MB/s). The architecture decision isn't "maximize swap size" — an oversized page file doesn't improve OneDrive sync, browser performance, or background processes, and it accelerates SSD wear for no benefit. The decision is: **size each tier deliberately against what actually needs to fit, and know exactly which workloads tolerate which tier's speed.**

- A model that fits entirely in the 26GB physical RAM tier (up to ~32B parameters, quantized) runs at interactive speed.
- A model that must spill into the 32GB NVMe swap tier (70B class, 4-bit) still runs — it does not crash — but at a fraction of the speed, because every token generation cycle has to shuttle weights across the slower storage lane.

## The three components, tuned independently

1. **WSL2's own memory/swap allocation** (`.wslconfig`) — governs what the Linux environment running the model server sees. Defaults to 50% of RAM and a small swap; must be set explicitly for AI workloads.
2. **The Windows page file** — a separate, system-level virtual memory mechanism, sized against the actual NVMe drive's real throughput rather than a generic "1.5x RAM" rule written for much slower disks.
3. **GPU VRAM (4GB)** — too small to hold a meaningful fraction of a 14B+ model's weights, so it's used for the first few layers only; the bulk of the work happens in the RAM/swap tiers above.

## What this replaces

Standing default advice is "buy a GPU with more VRAM" or "rent cloud GPU time." This architecture instead treats an already-owned, already-depreciated laptop as the substrate, and treats memory tiering — not model-shrinking, not new silicon — as the primary lever for how large a model that substrate can usefully run.
