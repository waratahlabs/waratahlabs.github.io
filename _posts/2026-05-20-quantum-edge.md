---
layout: post
title: "The Quantum Edge: Running QAOA on a Phone"
date: 2026-05-20
---

I shipped an iOS app to the App Store last week. 2MB. No cloud dependency. Runs quantum circuits natively on Metal.

The benchmark result that surprised me most: an M1 iPad Pro and an iPhone 16 Pro Max hit the same ceiling. Not close — identical. Both die at n=28 qubits with a 2GB statevector sitting in shared memory. n=29 kills the process with SIGKILL. No warning, no graceful exit — the kernel just disappears.

That's the honest version of where consumer quantum simulation sits in 2026. I'll get to the ceiling in a moment. First, what the app actually does.

---

## What shipped

The app runs four distinct quantum workloads on-device using a custom Metal compute kernel — `QuantumEdgeKit`, a Swift package that manages statevector state in `MTLBuffer` shared memory and dispatches gate operations as GPU compute shaders.

**The kernel is not a simulation of a simulation.** There's no intermediate representation, no classical emulation layer, no Python interpreter. Swift calls directly into Metal. The GPU dispatches the gate unitary. The statevector lives in memory the GPU already owns.

The workloads:

- **QPO circuit** — a depth-4 QAOA circuit for quantum prompt optimisation, pulled from a real QUBO matrix I computed in the QPO work earlier this year. Four qubits, Hadamard initialisation, cost and mixer layers, animated gate-by-gate with Bloch sphere readout via partial trace
- **Bloch sphere** — live single-qubit demo; gate buttons apply H, X, RZ(π/4) and watch the state vector rotate in real time via SceneKit
- **Bell state** — H(0)·CNOT correctness check; the app verifies the expected amplitudes against a tolerance and tells you whether the entanglement is right
- **Benchmark suite** — sweep n=4→20, find the ceiling (n=20→32), depth scaling n×d. Copy to clipboard with hardware specs embedded

The QPO tab runs the QAOA inner loop with fixed parameters derived from the QPO research — the variational circuit executing on Metal, gate by gate, with the statevector updating live. The outer parameter optimisation loop is hardcoded for this release; the planned follow-on is the full variational search using the Metal kernel as the inner oracle. That's when it stops being a demo and starts being a tool.

---

## Why Metal and not Accelerate

The M1 iPad Pro has the Accelerate framework — BLAS/LAPACK via the AMX coprocessor. I could have routed the statevector operations through NumPy-style matrix multiplication.

I didn't, for two reasons.

First, gate application on a statevector isn't a matrix multiply in the classical sense. Each single-qubit gate touches exactly half the statevector amplitudes — those where the target qubit bit is 0, and their paired partners where it's 1. That's a stride-access pattern that BLAS doesn't express naturally. Writing it as a dense matrix multiply wastes memory and compute.

Second, I wanted the compute to live on the GPU, not share the CPU with the app's UI thread. Metal compute shaders dispatch to the GPU and the statevector buffer never leaves GPU-accessible memory. The CPU just kicks off the dispatch and waits for the signal.

The tradeoff this creates is worth naming: Metal shaders work in float32, not the float64 that Qiskit and PennyLane use by default. For shallow QAOA circuits at depth 1–4, that's fine — float32 gives ~7 significant decimal digits per amplitude and errors don't accumulate meaningfully. I ran a cross-validation against PennyLane's `default.qubit` (float64) on the M1 Pro: GHZ and QAOA depth-4 circuits both show **max amplitude error ≤ 3e-07**, with fidelity 1.0000000000 across all tested circuits once qubit ordering conventions are aligned. The math is right. For fault-tolerant simulation or precision-sensitive research, you'd want float64 — but that halves the qubit ceiling, because the statevector doubles in size and the memory wall hits at n=27 instead of n=28.

The result is a kernel that does one thing and does it with no overhead. The 2MB install size is the proof: Swift compiles to native ARM64 machine code. No interpreter, no JIT, two hops from Swift to GPU ISA. PennyLane's NumPy dependency alone is bigger than the entire app.

---

## The ceiling

| Platform | Device | n=28 d=4 | vs Metal |
|----------|--------|----------|----------|
| Metal GPU | M1 iPad Pro (8GB) | ~9.3s | — |
| Metal GPU | iPhone 16 Pro Max A18 Pro (8GB) | ~9.3s | — |
| PennyLane CPU float64 | M1 Pro MacBook (32GB) | 384.5s | 44× slower |
| PennyLane CPU float64 | Predator U9-275HX (96GB) | TBD | — |
| PennyLane GPU CUDA float64 | RTX 5070 Ti Mobile | TBD | — |

The 8GB wall is the 8GB wall. A 28-qubit statevector is 2GB of complex floats (2^28 amplitudes × 8 bytes each). It fits. 29 qubits is 4GB. The OS kills it.

What's interesting is the M1 and A18 Pro are identical at n=28 depth 4. Both are memory-bandwidth-limited — the bottleneck isn't compute, it's moving 2GB through the memory subsystem on each gate application. The A18 Pro is a faster chip but it doesn't matter when the problem is bandwidth.

The M1 Pro CPU comparison is in now. At n=28 depth 4, PennyLane's `default.qubit` (float64) takes 384 seconds on the same machine where Metal finishes in 8.8 — **44× slower**. The crossover is at n=16: below that, CPU wins because Metal has dispatch overhead on small statevectors. Above n=16 Metal pulls away and doesn't look back.

This is the trade in concrete terms. Float64 on CPU is more precise and has no memory ceiling on a 32GB machine. Float32 on GPU is 44× faster at n=28 and hits a hard wall at n=29. For QAOA at the qubit counts that matter right now, the phone wins on throughput.

Depth scaling is linear: n=28 d=1→4 runs in roughly 1:2:3:4 ratio. No saturation. QAOA depth isn't the bottleneck — qubit count is. A depth-10 circuit on 28 qubits runs fine. A depth-1 circuit on 29 qubits doesn't.

The Predator and CUDA rows are pending — those determine where the ceiling sits with 96GB of RAM and a GPU that isn't sharing memory with the OS.

---

## The App Store bit

[Quantum Edge on the App Store](https://apps.apple.com/us/app/quantum-edge/id6770526340)

The submission got rejected on first pass — the review team saw a crash, and wanted a screen recording, device list, and feature walkthrough. Standard first-app friction for a technical tool without an obvious consumer use case.

The fix: build 2, a screen recording on a physical device, and a written feature description. The crash turned out to be a SwiftUI `@State` mutation happening during the render pass — one line in the SceneKit `UIViewRepresentable` wrapper calling a coordinator callback synchronously in `makeUIView`. Deferred with `Task { @MainActor in }`. Build 2 is live.

---

## What's next

The benchmark matrix has empty cells. Android (Pixel 9 Pro Fold) hasn't been benchmarked yet — PennyLane via pip in the Debian terminal is the approach, CPU-only, no GPU path available on Snapdragon without custom Vulkan work. The desktop platforms are next after that: M1 Pro CPU, Predator CPU, and the RTX 5070 Ti Mobile CUDA path. Full comparison table once all four are in.

The QPO outer loop is the follow-on that matters. What's live is the quantum inner loop running on Metal with fixed parameters. The real work is closing the variational loop — parameter search on the phone using the Metal kernel as the oracle. When that lands, the ceiling benchmark stops being a curiosity and the QPO result becomes actionable.

n=28. Everything above that is an open question.

The code — `QuantumEdgeKit`, the Metal kernels, the benchmark suite, the cross-validation scripts — will be open sourced shortly at [github.com/waratahlabs/quantum-edge](https://github.com/waratahlabs/quantum-edge).
