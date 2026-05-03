---
layout: post
title: "QPO: what QAOA actually does to your prompt candidates (n=50)"
date: 2026-05-03
---

Building agents at scale has two distinct problems that get conflated. The first is getting to a prompt that works at all — something that reliably handles the task for your use case. The second is making it good: iterating on real user feedback, handling the edge cases you didn't predict, tuning for the input distribution you only discover once the thing is in front of people. That second stage is genuinely craft, and it's irreplaceable. No eval suite you build before deployment captures the actual distribution of what users will type. Perfect day one is impossible by construction — the real eval set is the live session log.

QPO is about the first stage. That distinction matters when you're not building one agent. If you're building a platform where a thousand different people need to build a thousand different agents — each with their own goal, domain, and constraints — you can't rely on each of them having spent months honing their prompting instincts. Speed to good enough is the bottleneck. You need a way to navigate a large feature space and land somewhere reasonable, fast, across diverse problems you haven't seen before. The craft comes after. You need something to refine.

QPO is an attempt to push that cold-start search problem onto better-suited hardware. You have a space of possible prompt feature combinations — output format constraints, chain-of-thought instructions, role framing, exemplar count, domain context — and the question is whether QAOA (the Quantum Approximate Optimisation Algorithm) can find better starting points than a classical greedy scorer. This phase runs entirely on a CUDA-accelerated simulator (PennyLane `lightning.gpu`) — the goal is to validate the pipeline and understand where simulation shows promise before moving to physical quantum hardware.

---

## The Setup

Prompt feature selection is a natural fit for QUBO (Quadratic Unconstrained Binary Optimisation). Each feature is a binary variable: in or out. The cost function encodes quality. QAOA solves it by building a parameterised quantum circuit that uses interference effects to suppress low-quality combinations and amplify high-quality ones, then optimising the circuit parameters classically.

The pipeline has five stages: decompose the goal into features, generate candidate prompts from those features, pre-score each candidate with a small model, hand the top candidates to QAOA to shortlist a combination, deep-evaluate the result. The classical baseline is deliberately dumb: take the top-K by pre-score rank, no combinatorial reasoning at all.

For this run: QAOA simulated on PennyLane `lightning.gpu` (CUDA-accelerated statevector simulator on a local Linux box — not physical QPU hardware), pre-scoring and deep evaluation on `granite4.1:3b` and `granite4.1:8b` on Mac Ollama. `circuit_depth=3`, `num_iterations=50`, 20 qubits.

---

## Fifty Runs, Five Goals

| Goal | QAOA wins | Classical wins | Ties | Mean Δ |
|------|-----------|----------------|------|--------|
| JSON extraction | 0 | 1 | 9 | −0.005 |
| Ticket classification | 1 | 0 | 9 | +0.005 |
| **Git commit message** | **5** | **1** | **4** | **+0.030** |
| CVE risk assessment | 1 | 2 | 7 | −0.005 |
| Legal clause rewrite | 1 | 1 | 8 | 0.000 |
| **Total** | **8** | **5** | **37** | **+0.005** |

8 QAOA wins. 5 classical wins. 37 ties.

The headline number is the easy part. The more interesting thing is where the wins come from. The git commit task accounts for 5 of the 8 QAOA wins, with deltas up to +0.10. Every other task is close to a wash. Ties dominate at 74%, which is the honest summary of what Phase 1 diagonal QUBO actually does — most of the time QAOA and greedy find each other.

I ran a single-pass preliminary before this batch with CVE as the standout result (+0.05). At n=10, CVE goes 1 QAOA win, 2 classical wins. The preliminary read was a stochastic outlier.

---

## What The Data Actually Says

The short version: Phase 1 QPO uses a diagonal QUBO, and a diagonal QUBO is the problem class where classical greedy search is provably optimal.

A diagonal QUBO means each feature's contribution to the cost function is computed independently — no cross-terms. If quality is a sum of independent feature contributions, the globally optimal subset is exactly the top-K by individual score. Greedy top-K doesn't approximate the optimal solution; it *is* the optimal solution.

QAOA running over a diagonal cost landscape will converge to roughly the same shortlist as greedy. Any wins are a stochastic divergence — QAOA taking a different trajectory through the same flat landscape — not structural advantage.

Which makes the git commit result worth looking at more carefully. A conventional commit message has no single correct output. The question is which combination of prompt features — how much context to include, how to frame the imperative, whether to include scope, what level of abstraction to describe the change at — produces the best result for a given diff. The answer isn't fixed; many combinations work and they work through different paths. The greedy scorer ranks features independently. QAOA's trajectory occasionally finds a combination the greedy scorer underweighted, and on this task those combinations consistently produce better outputs. On constrained-output tasks (JSON, classification), the answer space is narrow and the greedy optimum and the combinatorial optimum converge. There's nowhere to diverge.

CVE looks like it should follow the git commit pattern — complex, multi-factor, open-ended output. It doesn't. Average candidate overlap is 6.6/10 across the CVE runs, the highest of any task, which suggests the pre-scorer is doing a better job of ranking CVE features than it is with commit message features. The stochastic divergence that showed up in the preliminary was just that.

---

## What Comes Next

Phase 1 diagonal QUBO is, by construction, the regime where this comparison is hardest for QAOA. The natural progression has three steps.

**Remove the pre-filter.** The pipeline currently pre-filters to the top-20 candidates by pre-score before QAOA sees anything. That's a greedy gate before the quantum stage — it eliminates candidates by individual score before QAOA can evaluate them as combinations. If the pre-scorer discards a feature that only scores well in combination with another, QAOA never sees it. The test: raise the filter cap, let QAOA work the full generated space, measure whether win rate on open-ended tasks like git commit increases. If it does, the pre-filter was the ceiling, not the circuit depth.

**Scale the simulation.** 20 qubits covers 20 candidate features. Moving to 50 qubits starts approaching the practical limits of classical statevector simulation — at 50 qubits, the state space is 2⁵⁰, which is where GPU memory constraints on even large CUDA systems become binding. That's a meaningful threshold: it's the regime where simulating the circuit classically stops being cheap, which is also the regime where running it on real hardware starts to be the only practical option.

**Move to physical hardware.** Once the pipeline is validated at scale on the simulator and the QUBO structure is well-characterised, the interesting question becomes what NISQ noise does to the result distribution. Does hardware noise wash out the advantage on open-ended tasks, or does the signal persist? That experiment hasn't been run anywhere in the literature for this problem class. The answer — in either direction — is worth knowing.

The writeup committed at the start to publishing regardless of direction. The finding is: 8 QAOA wins, 5 classical wins, 37 ties, advantage concentrated in open-ended compositional tasks, diagonal QUBO is the structural limit. That's what the data shows.

Code: [github.com/waratahlabs/qpo](https://github.com/waratahlabs/qpo).
