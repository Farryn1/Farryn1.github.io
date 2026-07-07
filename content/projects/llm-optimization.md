+++
title = "LLMs as Optimizers — Reproducing OPRO"
date = "2026-06-27"
slug = "llm-optimization"
tags = ["LLM", "optimization", "OPRO", "reproducibility"]
icon = "fa-solid fa-robot"
accent = "#7048e8"
summary = "Reproducing the OPRO paradigm (LLMs as optimizers) on simulated tasks, benchmarked against classical search under a matched evaluation budget."
+++

A small, self-contained research project that reproduces the **OPRO** paradigm from *Yang et
al., 2023, "Large Language Models as Optimizers"* on two simulated optimization tasks, and
compares it against classical baselines. It **runs end-to-end with or without an API key** — a
deterministic *mock* backend guarantees the entire pipeline is reproducible offline, while a
real Claude backend is used when `ANTHROPIC_API_KEY` is set.

<!--more-->

<a class="download-btn" href="https://github.com/Farryn1/llm-opro-optimizer" target="_blank" rel="noopener"><i class="fa-brands fa-github"></i> View source on GitHub</a>

## What is OPRO?

Classical optimizers (gradient descent, evolutionary algorithms, 2-opt, …) update a candidate
solution with hand-designed rules. **OPRO** replaces the update rule with a **large language
model**. The optimization state is described *in natural language* inside a **meta-prompt**; the
LLM reads it and **proposes the next candidate**; that candidate is evaluated by the black-box
objective; and the new `(solution, score)` pair is appended to the history. Iterating lets the
LLM act as a general-purpose, derivative-free optimizer that "learns" the landscape from the
trajectory it is shown.

```
      ┌──────────────────────────────────────────────┐
      │  Meta-prompt: task description + top-k         │
      │  (solution -> score) pairs, sorted best-last   │
      └───────────────────────┬──────────────────────┘
                              │  propose()
                              ▼
                ┌───────────────────────────┐
                │  LLM (Claude) / mock       │  → new candidate solution
                └───────────────┬────────────┘
                                │  parse (robust regex)
                                ▼
                    evaluate objective f(·)
                                │
                                ▼
        append (solution, score) to trajectory → repeat
```

### The meta-prompt mechanism

Each round the prompt contains (1) a short **task description**; (2) the **top-k trajectory**,
sorted *worst-to-best* so the strongest examples appear last (LLMs anchor on recent context);
and (3) an explicit **output-format instruction**. Replies are parsed with robust regular
expressions, with a **random-valid fallback** on parse failure so the loop never stalls.

## Tasks & baselines

Two simulated tasks — **black-box continuous minimization** (Rastrigin 2-D, Ackley 5-D,
Rastrigin 10-D) and a **small Traveling Salesman Problem** — are each compared against baselines
under an **identical evaluation budget**:

| Task        | Baselines                                   |
|-------------|---------------------------------------------|
| Continuous  | Random search; hill-climbing local search   |
| TSP         | Random restart; 2-opt local search          |

## Reproducibility: the mock backend

The deterministic mock (`MockProposer`) *imitates* a competent LLM optimizer while requiring no
network: for continuous tasks it perturbs the best-so-far point with a **shrinking step size**
(simulated-annealing flavor) or extrapolates between the two best points; for TSP it applies a
**2-opt segment reversal**. This keeps the pipeline runnable and results reproducible — it is a
stand-in, not a claim about LLM performance. Enabling the real Claude Messages API
(`--backend auto`, `DEFAULT_MODEL = "claude-sonnet-4-6"`) yields genuine LLM proposals.

## Results

Convergence figures show OPRO's best-so-far curve **decreasing monotonically over rounds**,
demonstrating that the LLM-as-optimizer loop (even with the deterministic mock) makes steady
progress and is competitive with classical search under an identical evaluation budget.

## Reference

Yang, C., Wang, X., Lu, Y., Liu, H., Le, Q. V., Zhou, D., & Chen, X. (2023).
*Large Language Models as Optimizers.* [arXiv:2309.03409](https://arxiv.org/abs/2309.03409).

*Stack: Python · Anthropic SDK · NumPy · Matplotlib · Pandas*
