# Results — What Actually Runs, and How Well

Measured on: Dell G7 7588, i7-8750H (12 threads), 32GB RAM, GTX 1050 Ti 4GB VRAM, 58GB functional memory pool (26GB WSL2 physical RAM + 32GB NVMe virtual swap).

| Model class | Example | Download size | Memory footprint | Where it loads | Speed | Frontier baseline (qualitative) |
|---|---|---|---|---|---|---|
| 14B / 15B | DeepSeek-R1 14B | ~9GB | ~9GB | 100% physical RAM | ~20–25 tok/s | Claude 3.5 Haiku / GPT-4o-mini class |
| 32B | Qwen 2.5 Coder 32B, DeepSeek-R1 32B | ~20GB | ~20–23GB | 100% physical RAM | ~10–15 tok/s | Claude 3.5 Sonnet / Gemini 1.5 Pro class (esp. coding/logic) |
| 70B / 72B (4-bit, Q4_K_M) | Llama 3.1 70B, Qwen 2.5-72B-Instruct | ~40–43GB | ~42–45GB (before context overhead) | RAM + ~24GB NVMe swap | ~0.5–1.5 tok/s | Llama 3.1 70B / GPT-4 Turbo class (capability), but unusable interactively at this speed |

## Reading these numbers correctly

- **14B is the "always works" tier** — fast enough to feel like a real conversation, ships in ~9GB, no swap involvement.
- **32B is the daily-driver sweet spot on this specific hardware** — it's the largest class that fits entirely in the 26GB physical RAM tier, so it never touches the ~7x-slower NVMe swap lane. This is where the machine's capability-per-second is actually highest.
- **70B is a ceiling test, not a workflow.** It proves the memory architecture holds — the model loads and generates without an out-of-memory crash — but at 0.5–1.5 tokens/sec it is not usable for anything requiring back-and-forth interaction. Treat it as evidence of what's *possible*, not a recommendation.

## The qualitative baseline comparisons

Frontier-model comparisons above are qualitative positioning (task-class capability), not benchmark scores — they describe roughly which tier of commercial model a given local model's reasoning/coding quality is comparable to, not a claim of matching exact benchmark numbers.

## What this means for capacity planning

If you're sizing a similar setup: budget physical RAM for the largest model class you want as your *daily driver*, not your absolute ceiling. The swap tier's job is to let a larger model load without crashing when you deliberately want to test the limit — not to be where your everyday workload lives.
