---
layout: page
title: projects
permalink: /projects/
---

## Quantum Edge

Portable quantum simulation and **quantum prompt optimisation (QPO)** — run real quantum circuits, on the device in your pocket, without a cloud dependency, and find out exactly where it breaks.

*[Full project page — QPO explained, benchmarks, FAQ →](/project/quantum-edge/)*

App Store: [Quantum Edge](https://apps.apple.com/us/app/quantum-edge/id6770526340)  
Code: open source — repo publishing shortly.

*Status: iOS live on App Store — Android and desktop benchmarks in progress*

---

## QPO — Quantum Prompt Optimisation

QUBO/QAOA formulations for prompt feature space search. Prompt quality is a combinatorial problem — feature interactions determine outcomes in ways linear scoring can't capture. QPO formulates feature selection as a QUBO and solves it with QAOA, running the combinatorial search on gate-based quantum hardware rather than greedy approximation.

Preliminary findings: QAOA advantage is task-complexity-gated. On constrained-output tasks (JSON, classification), QAOA and classical greedy converge to the same shortlist. On complex multi-factor tasks (CVE risk assessment), QAOA found a higher-scoring candidate the greedy scorer missed. The mechanistic explanation — and the limit condition for Phase 1 — is in the writeup.

Phase 1 ran on a CUDA-accelerated simulator. Phase 2 extended to larger circuits (50 qubits) and published results. Phase 3 — physical quantum hardware — is in planning. Code: [github.com/waratahlabs/qpo](https://github.com/waratahlabs/qpo).

*[Phase 1 results (n=50) →](/2026/05/03/qpo-preliminary-findings/)*
*[Phase 2 results →](/2026/05/05/qpo-phase2/)*
*[Preprint on Zenodo →](https://doi.org/10.5281/zenodo.20394090)*

*Status: preprint published (Zenodo, 2026-05-26)*
