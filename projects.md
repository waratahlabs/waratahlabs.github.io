---
layout: page
title: projects
permalink: /projects/
---

## QPO — Quantum Prompt Optimisation

QUBO/QAOA formulations for prompt feature space search. Prompt quality is a combinatorial problem — feature interactions determine outcomes in ways linear scoring can't capture. QPO formulates feature selection as a QUBO and solves it with QAOA, running the combinatorial search on gate-based quantum hardware rather than greedy approximation.

Preliminary findings: QAOA advantage is task-complexity-gated. On constrained-output tasks (JSON, classification), QAOA and classical greedy converge to the same shortlist. On complex multi-factor tasks (CVE risk assessment), QAOA found a higher-scoring candidate the greedy scorer missed. The mechanistic explanation — and the limit condition for Phase 1 — is in the writeup.

Phase 1 ran on a CUDA-accelerated simulator. Phase 2 extended to larger circuits (50 qubits) and published results. Phase 3 — physical quantum hardware — is in planning. Code: [github.com/waratahlabs/qpo](https://github.com/waratahlabs/qpo).

*[Phase 1 results (n=50) →](/2026/05/03/qpo-preliminary-findings/)*
*[Phase 2 results →](/2026/05/05/qpo-phase2/)*

*Status: Phase 2 complete — Phase 3 in planning*

