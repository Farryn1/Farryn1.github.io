+++
title = "Large-Scale Facility Location: MILP vs. Benders vs. Lagrangian"
date = "2026-06-28"
slug = "large-scale-optimization"
tags = ["optimization", "MILP", "Benders decomposition", "Lagrangian relaxation"]
icon = "fa-solid fa-diagram-project"
accent = "#1565c0"
summary = "Solving the capacitated facility location problem three ways — direct MILP, Benders, and Lagrangian relaxation — and comparing them at scale."
+++

A self-contained project that solves the **Capacitated Facility Location Problem (CFLP)** three
ways — direct MILP, Benders decomposition, and Lagrangian relaxation — and compares them on
synthetic instances of growing size. A compact showcase of large-scale optimization practice,
using **only open-source solvers** (CBC via PuLP, HiGHS via SciPy); no commercial license
required.

<!--more-->

<a class="download-btn" href="https://github.com/Farryn1/facility-location-decomposition" target="_blank" rel="noopener"><i class="fa-brands fa-github"></i> View source on GitHub</a>

## Motivation

Facility location underpins supply-chain network design, warehouse and data-center placement,
emergency-service siting, and telecom planning. The capacitated version is **NP-hard**, and
real instances are large. A monolithic MILP quickly becomes intractable, so practitioners turn
to **decomposition**: split the hard "which facilities to open" decision from the easy "how to
serve demand" LP, and iterate.

## Problem formulation

- Facilities $j$ with opening cost $f_j$ and capacity $s_j$; customers $i$ with demand $d_i$.
- Transport cost $c_{ij}$ = unit cost × Euclidean distance.
- $y_j \in \{0,1\}$: open facility $j$; $x_{ij} \in [0,1]$: fraction of customer $i$'s demand
  served by facility $j$ (splittable transportation formulation, so the assignment block is an LP).

$$
\min \ \sum_j f_j y_j + \sum_i \sum_j c_{ij} d_i x_{ij}
$$

subject to full-demand service ($\sum_j x_{ij} = 1$), capacity-and-linking
($\sum_i d_i x_{ij} \le s_j y_j$), and $x_{ij} \le y_j$.

## The three methods

**Direct MILP** — builds the full model in PuLP and solves with the bundled CBC
branch-and-bound solver. Ground-truth optimum where tractable.

**Benders decomposition** — a **master MILP** over $y$ plus an auxiliary variable $\theta$
under-estimating transport cost, augmented by a global capacity cut. For fixed $y$, the
transportation **LP subproblem** yields the true transport cost and, through its dual,
**optimality cuts** (or **feasibility cuts** from dual extreme rays when demand cannot be
served). The master objective is a global **lower bound**; each feasible $y$ gives an **upper
bound**; iterate until the gap closes. Duals come from SciPy `linprog` (HiGHS).

**Lagrangian relaxation** — relaxes the demand equalities into the objective with multipliers
$\lambda_i$. The relaxed problem **decomposes per facility** into a cheap open/close +
continuous-knapsack decision, giving a valid lower bound $L(\lambda)$. Multipliers are updated
by the **subgradient method** with a Polyak step; each iteration also repairs a feasible upper
bound.

## Why decomposition helps at scale

The monolithic MILP has $n + mn$ variables and $O(mn)$ linking constraints; branch-and-bound
explodes as $m, n$ grow. **Benders** keeps only the $n$ hard binaries in the master and pushes
the $mn$ continuous variables into a fast LP. **Lagrangian** breaks the coupled problem into
$n$ tiny per-facility subproblems solved in closed form. Both turn one huge problem into many
small ones plus a coordinating loop — the central idea of large-scale optimization.

## Results

{{< figure src="/images/projects/opt_benders_convergence.png" alt="Benders convergence" caption="Benders upper- and lower-bound curves closing to the optimum." >}}

{{< figure src="/images/projects/opt_lagrangian_convergence.png" alt="Lagrangian convergence" caption="Lagrangian lower bound rising toward a feasible upper bound." >}}

{{< figure src="/images/projects/opt_runtime_scaling.png" alt="Runtime scaling" caption="Runtime vs. instance size for all three methods." >}}

{{< figure src="/images/projects/opt_optimality_gap.png" alt="Optimality gap" caption="Each method's optimality gap relative to the MILP optimum." >}}

Typical observations: **Benders** converges to the exact optimum (gap → 0); the **direct MILP**
matches it on small/medium sizes but scales worse; **Lagrangian** returns a tight bound and a
near-optimal feasible solution quickly, illustrating the accuracy/speed trade-off.

*Stack: Python · PuLP (CBC) · SciPy (HiGHS) · NumPy · Matplotlib*
