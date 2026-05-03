---
layout: page
title: projects
permalink: /projects/
---

## Speculatores

Portable AI agent runtime for network reconnaissance. Inference-native C2 — the agent reasons about findings at the edge in real time, compressing time-to-exploit-discovery into a single on-device pass.

*Status: active development*

---

## QPO — Quantum Prompt Optimisation

QUBO/QAOA formulations for prompt feature space search. Prompt quality is a combinatorial problem — feature interactions determine outcomes in ways linear scoring can't capture. QPO formulates feature selection as a QUBO and solves it with QAOA, running the combinatorial search on gate-based quantum hardware rather than greedy approximation.

Preliminary findings: QAOA advantage is task-complexity-gated. On constrained-output tasks (JSON, classification), QAOA and classical greedy converge to the same shortlist. On complex multi-factor tasks (CVE risk assessment), QAOA found a higher-scoring candidate the greedy scorer missed. The mechanistic explanation — and the limit condition for Phase 1 — is in the writeup.

Phase 1 runs on a CUDA-accelerated simulator. Next phases: larger circuits (50 qubits, approaching classical simulation limits), then physical quantum hardware. Code: [github.com/waratahlabs/qpo](https://github.com/waratahlabs/qpo).

*[Results (n=50) →](/2026/05/03/qpo-preliminary-findings/)*

*Status: Phase 1 complete — results published*

---

## KYC React Native Library

On-device identity verification for React Native. NFC ePassport reading, OCR, face match — all on-device, no PII transmitted. The institution gets a proof, not the document.

*Status: active development*
