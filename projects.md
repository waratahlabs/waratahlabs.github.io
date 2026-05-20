---
layout: page
title: projects
permalink: /projects/
---

## Quantum Edge

Portable quantum simulation — run real quantum circuits on the device in your pocket, without a cloud dependency, and find out exactly where it breaks.

The core is `QuantumEdgeKit`, a Metal compute kernel written in Swift. Statevector lives in `MTLBuffer` shared memory. Gate operations are GPU compute shaders. No intermediate representation, no Python interpreter, no JIT. Swift calls Metal directly.

The iOS app ships four workloads: a QAOA circuit drawn from the QPO research, a live single-qubit Bloch sphere demo, a Bell state correctness check, and a benchmark suite that sweeps qubit count until the process dies. On any 8GB Apple device — M1 iPad Pro, iPhone 16 Pro Max — the ceiling is n=28. A 28-qubit statevector is 2GB of complex floats. n=29 is 4GB; the OS kills the process with SIGKILL, no warning. At n=28 depth 4, the Metal kernel finishes in 8.8 seconds. The same circuit on PennyLane's float64 CPU backend takes 384 seconds on the same machine.

The app is 2MB installed. PennyLane's NumPy dependency alone is larger.

*[The full writeup, benchmark data, and cross-validation methodology →](/2026/05/20/quantum-edge/)*

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

*Status: Phase 2 complete — Phase 3 in planning*
