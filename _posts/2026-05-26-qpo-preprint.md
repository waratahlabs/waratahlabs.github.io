---
layout: post
title: "QPO: preprint on Zenodo"
date: 2026-05-26
---

The QPO paper is now published as a preprint. DOI: [10.5281/zenodo.20394090](https://doi.org/10.5281/zenodo.20394090).

It brings together everything from the first ideation a month ago through to the AWS Batch classification experiments that wrapped up last week: four experiments across 480 runs, a 2×3 factorial study on cloud GPU, and a parallel on-device implementation running on Apple silicon.

---

## The results

A diagonal QUBO baseline (20 qubits, n=50) establishes a 16% QAOA win rate over classical greedy. Removing the pre-filter cap doesn't improve it: the ceiling is the formulation, not the filter. Adding off-diagonal ZZ cross-terms raises the win rate to 21% (24 qubits, n=100), distributed across 9 of 10 task types. YAML CI config is the exception: a constrained structured output task where greedy already finds the ceiling and cross-terms add noise rather than signal.

A 2×3 factorial experiment (300 runs, cloud GPU) tests whether advantage concentrates by task type and prompt depth. Task type: confirmed. Open compositional tasks produce an 18% QAOA win rate against 3% classical. Depth: not so much. Shallow prompts show marginally higher advantage than deep, the opposite of the predicted direction. The pre-scorer returned near-uniform scores across candidates in that experiment, which flattened the QUBO landscape and left QAOA without a gradient to exploit.

The on-device variant runs the full pipeline on iPhone using Apple Intelligence and the Accelerate statevector simulator. Axis-encoding rather than candidate-encoding; exact Q_ij derived within each run; no cloud dependency. The pipeline completes a 100-point sweep in under 2.5 seconds on an iPhone Air.

---

## Where we didn't explore

All results are simulation-based. The 21% win rate is against a statevector simulator: an exact classical representation of quantum state, with no gate noise or decoherence. Whether the signal survives the transition to physical QPU hardware is the open question. That's Phase 4 -- Braket or on-device accumulation of cross-session co-occurrence structure.

---

## Why publish now

QUBO and LLM interactions are an evolving area. By publishing the preprint in all its experimental messiness, the core thesis is in the record: the simulation signal exists, the task-type hypothesis is supported, and the on-device variant runs end-to-end. Future work can improve and iterate on that foundation -- stronger baselines, better scorer calibration, hardware validation -- rather than starting from scratch.

Code and data will be released in a future update.

---

*Hargan, W. (2026). Quantum Prompt Optimisation: QUBO/QAOA-Based Feature Selection for LLM Output Quality. Zenodo. [https://doi.org/10.5281/zenodo.20394090](https://doi.org/10.5281/zenodo.20394090)*
