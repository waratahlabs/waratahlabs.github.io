---
layout: post
title: "Is 2x Sustained Performance Just Hype? A18 Pro vs A20 Pro, Benchmarked"
description: "A20 Pro vs A18 Pro, benchmarked: 27% faster cold, 10–15% slower once throttled, and 2x faster on-device AI with an occasional 400-second stall."
date: 2026-09-21
---

*Benchmark harness (Metal/CUDA/CPU quantum-simulation comparisons): [github.com/waratahlabs/quantum-metal-bench](https://github.com/waratahlabs/quantum-metal-bench)*

I pulled the export off the iPhone 18 Pro expecting a clean sweep. What I got was one row that took four hundred seconds.

Not a crash, not a freeze: a single scoring call, on a prompt the same phone had handled in under twenty seconds ninety seconds earlier, in the middle of a run it was otherwise winning. Same prompt set on both phones, same fixed workload, a chip with double the Neural Engine cores of the phone next to it, and one specific task took twenty times longer than its own baseline.

That's not the story Apple is telling this week. Apple's own launch-event comparison chart puts it plainly: "Sustained performance," iPhone 18 Pro as the baseline, iPhone 17 Pro at "40% more," iPhone 16 Pro at "2x more," captioned on screen as "compared to iPhone 16 Pro and older." Add a 2-nanometer process shrink and 50% more memory bandwidth over the previous generation, and that's a real, confirmed set of numbers, not a rumor or a misquote. I wanted to know what they look like on a workload that isn't Apple's own benchmark suite: a quantum-inspired optimisation pipeline I built, called Quantum Edge, that hammers both the GPU (a Metal statevector simulator) and the on-device Foundation Model (Apple's local LLM, scoring candidate outputs) in the same run.

2x sustained performance sounds like a spec. It isn't one until you name the workload, the thermal condition, and the chassis it was measured in. Here's what ten loops of identical work on both phones actually showed.

---

## The chip Apple didn't put in my hands

Before the numbers: a disclosure that matters more than it looks like it should.

Apple's own chart names its comparison device precisely: "iPhone 16 Pro," not "iPhone 16 Pro Max." Both sides of Apple's claim are the smaller, non-Max chassis. I don't have an 18 Pro Max. I didn't preorder, and by the time I went looking it was gone, so my new-phone side matches Apple's test article exactly. My old-phone side doesn't: I'm running the bigger iPhone 16 Pro Max against Apple's chosen 18 Pro, which means the three-year-old chip in this comparison gets more body and more thermal mass to work with than the phone Apple actually measured against.

That's a real methodological gap, and it cuts in the opposite direction you might expect. It doesn't hand the new chip an unfair fight. It hands the old chip a thermal-mass advantage Apple's own comparison never gave it. Any generational win the A20 Pro pulls off here, it pulls off against a tougher opponent than Apple's chart used.

| | A18 Pro (16 Pro Max) | A20 Pro (18 Pro) |
|---|---|---|
| Process | TSMC 3nm | TSMC 2nm (Apple's first) |
| GPU cores | 6 | 7 |
| Memory | 8GB, ~17% bandwidth bump over A17 Pro | 12GB, 96-bit LPDDR5x, 50% more bandwidth than A19 Pro |
| Neural Engine | 16-core | 32-core |
| Chassis (this test) | Pro Max | Pro |
| Chassis (Apple's own comparison) | Pro | Pro |

Every result below carries that asterisk. Where I can separate "this is the chip" from "this is the smaller body," I do. Where I can't, I say so.

One more piece of context before the numbers: the GPU kernel doing this work isn't Quantum Edge's alone. It's the same Metal statevector engine underneath [quantum-metal-bench](https://github.com/waratahlabs/quantum-metal-bench), the cross-platform benchmark harness we've been building to compare quantum-simulation performance across Metal, CUDA, and CPU backends. Quantum Edge is that engine running on a phone instead of a workstation. That matters for how to read what follows: a phone in your pocket, running this workload continuously until it throttles, was never the target use case for either project. What the cooled and cold-start numbers actually show is the raw architectural headroom Apple's silicon has to offer a statevector simulator. The throttled numbers show what a phone's thermal envelope does to that headroom when you refuse to stop asking for it.

---

## Cold, the newer chip wins. Comfortably. Not by 50%.

I ran the pipeline uncooled on both phones and pulled the very first loop, before either device had built up any heat. This is the number closest to what a spec sheet promises.

| | 16 Pro Max, loop 0 | 18 Pro, loop 0 |
|---|---|---|
| QAOA compute, median | 19.4ms | 14.1ms |
| QAOA compute, max | 30ms | 29.5ms |

**The A20 Pro is 27% faster cold.** That's a real generational jump, and it's the only clean win I found for the new chip on GPU compute. It's also less than half of the 50% bandwidth increase Apple advertises, and a long way short of "2x." Cold-start, best-case, favourable framing: 27%.

---

## Cooled to a standstill, they're the same phone

I ran both devices through the same pipeline with a Peltier cooler holding thermal state near baseline throughout. This is about as close to "compare the silicon, not the packaging" as I could get without a lab.

| | 16 Pro Max, cooled | 18 Pro, cooled |
|---|---|---|
| QAOA compute, median range across loops | 14.7–15.3ms | 12.4–19.1ms |
| QAOA compute, max | 30–49ms | 27–38ms |

They tied. No clean winner. If you artificially remove heat as a variable, two chips separated by a full node shrink and an extra GPU core come out roughly even on this workload. The 18 Pro's own range is wide enough (12.4 to 19.1ms) to fully overlap the older phone's tighter one.

That's the first sign that whatever the A20 Pro's real advantage is, it isn't sitting in raw arithmetic throughput on this kind of kernel. It's sitting somewhere thermal.

---

## Let them get hot, and the old phone wins

This is the result that made me go back and re-check my own methodology twice, because it's the opposite of what a "2x sustained performance" claim should produce.

Uncooled, both phones reach maximum thermal state at almost exactly the same point, around loop 1. Onset timing is a wash. What differs is where they land.

| | 16 Pro Max, throttled steady-state | 18 Pro, throttled steady-state |
|---|---|---|
| QAOA compute, median | 18.7–21.8ms | 21.9–23.7ms |
| QAOA compute, max | 28.9–36.7ms | 34.3–42ms |

**Fully throttled, the three-year-old chip in the bigger phone is 10–15% faster than the brand new one.** Cold, the A20 Pro wins by 27%. Hot, it loses by 10–15%. The generational advantage doesn't just shrink under sustained load. It inverts.

My working theory is the chassis asymmetry I flagged at the top: the A20 Pro may well have more raw headroom than the A18 Pro, but the smaller Pro body gives it less mass to sustain that headroom against. A chip with more to give and less room to cool it in can end up worse off than an older chip in a bigger box, once the thermal budget actually runs out. I can't fully separate "smaller chassis" from "different chip" with the hardware I have, which is exactly why the 18 Pro Max comparison still needs to happen.

---

## All four conditions, side by side

Put the three GPU comparisons next to each other and the pattern is a straight line from "new chip wins" to "new chip loses." Nothing else changes except how much heat was in the room.

| Condition | 16 Pro Max median | 18 Pro median | Winner |
|---|---|---|---|
| Cooled (thermal held near baseline) | 14.7–15.3ms | 12.4–19.1ms | Tie |
| Uncooled, cold start (loop 0) | 19.4ms | 14.1ms | 18 Pro, +27% |
| Uncooled, throttled steady-state | 18.7–21.8ms | 21.9–23.7ms | 16 Pro Max, +10–15% |

Three ways of running the identical pipeline, three different verdicts on which chip is faster. None of them is "wrong": they're measuring three different thermal regimes, and "2x sustained performance" doesn't say which one it means. Apple didn't run this benchmark, so I can't tell you which of these three numbers its slide corresponds to. What I can tell you is that at least one of them, the one that matters most for a phone actually being used, doesn't say what the marketing implies.

![QAOA compute time, min/median/max, across cooled, cold-start, and throttled conditions](/assets/images/a20-pro-qaoa-thermal-conditions.png)

Watch the dot, not the whiskers: the median flips sides across the three panels. Tied, then 18 Pro, then 16 Pro Max, in that order, on the same chip, on the same workload.

---

## The on-device model is twice as fast, until it isn't

Every loop in this run also scores candidate outputs through Apple's on-device Foundation Model. I fixed the number of scoring candidates at 32 on both phones specifically so this comparison wouldn't be muddied by one phone doing more LLM calls than the other. With that controlled, the Neural Engine story looked clean at first.

| | 16 Pro Max median | 18 Pro median |
|---|---|---|
| Loop 0 | 37.3s | 11.3s |
| Overall | 44.0s | 22.4s |

**Twice as fast, consistently, for the entire run.** Doubling the Neural Engine core count, 16 to 32, shows up here exactly like you'd expect it to. This is the generation's actual headline win, and it's a real one.

Then I looked at the tail.

| | 16 Pro Max, max observed | 18 Pro, max observed |
|---|---|---|
| Whole run | 73.3s | 408.3s |

The 16 Pro Max never went above 73 seconds on any of 29,900 scoring calls across the full run. The 18 Pro spiked past 400 seconds, more than five times its own median and five and a half times the old phone's worst case, on the phone with the faster chip.

![On-device Foundation Model scoring time, min/median/max, log scale](/assets/images/a20-pro-prescore-spread.png)

Note the log scale. That whisker isn't a rounding artifact. The 18 Pro's best-case number is genuinely better than the 16 Pro Max's. Its worst case is a different order of magnitude.

---

## Two prompts, and only two, break the new phone

I went looking for a pattern in the spikes and found one immediately: every single spike over 150 seconds on the 18 Pro traced back to exactly 2 prompts out of the 30 in the set. Both ask the model to generate real prose: one is a product pitch, the other is a Slack thread rewritten as a formal RFC. Long-form generation, not short scoring.

The obvious explanation, that long-form generation is just expensive, full stop, doesn't survive contact with the 16 Pro Max's data. I pulled the same two prompts from the older phone's run. Zero spikes. Zero of 2,000 rows over 60 seconds. On the 16 Pro Max, these are two unremarkable prompts sitting in the middle of the pack.

**This is a failure mode that exists on the new chip and does not exist on the old one, triggered by exactly the same input.**

The 18 Pro carries 12GB of RAM against the 16 Pro Max's 8GB. Apple has already confirmed, in writing, that iOS 27's most advanced on-device AI tier requires a 12GB minimum to run at all. I can't prove the 18 Pro is running a different, larger Foundation Model tier than the 16 Pro Max. Apple doesn't expose that from the API I'm calling, and I don't have the instrumentation to check model weights or memory footprint directly. But the mechanism is directly analogous to something Apple is already confirmed to be doing this generation: gating a bigger on-device AI capability behind the extra RAM. A larger model would explain both halves of this result in one shot: faster median (more capable, more compute to throw at it), and worse tail behaviour under memory and thermal pressure during the specific calls that ask for a lot of tokens back.

That's a hypothesis, not a finding. I'm flagging it as the open question, not the answer.

---

## What actually happened when I stopped watching the graphs

The whole ten-loop run finished roughly an hour faster on the 18 Pro than on the 16 Pro Max. In aggregate, over a long session, the newer phone is meaningfully quicker: the median wins are real and they compound.

But "faster on average" and "predictable" are different properties, and only one of them showed up on the new chip. A person prompting for something that needs a real written answer, on a phone that's already warm in their pocket, could sit through a stall the three-year-old phone next to them simply would not produce. That's not a number Apple puts on a slide, and it's not a number you'd find unless you built a fixed, repeatable workload and ran it until something broke the pattern.

---

## What I still don't know

I'm not running an 18 Pro Max, so I can't tell you how much of the throttled-state loss is chip and how much is chassis. That comparison needs the Pro Max, and I plan to revisit it once the launch hype dies down and I can actually source one. I'm inferring the model-size explanation for the tail latency from a RAM number and an unrelated Apple disclosure about Siri, not from anything Apple has said about the Foundation Models framework specifically. And I only found two prompts that trigger the spike in a 30-prompt set. There may be a third or fourth pattern I haven't isolated yet, or a fix already shipping in a later iOS point release.

One more limitation worth stating plainly: this is one phone per generation, not a batch. Chip binning means two A20 Pro units can behave slightly differently under identical thermal load. I can't tell you if my 18 Pro is a representative unit or one end of a distribution.

Cold, the A20 Pro is the better chip by a real but modest margin. Hot, it isn't. And somewhere in that extra 4GB of RAM is a faster average and an occasional four-minute wait that the old phone doesn't know how to produce.

None of that is a verdict on the architecture itself. The cooled and cold-start numbers are the closer read on what the A20 Pro's silicon can actually do. A phone getting hot in your hand and slowing down is a phone problem, not a chip problem. But "sustained" is doing a lot of work in that marketing line, and I'd like Apple to say, in public, which of my three tables it's standing behind. I have a guess about which one it isn't.

---

## Frequently asked questions

### Is the A20 Pro faster than the A18 Pro?

Cold, yes: in my tests the A20 Pro (iPhone 18 Pro) was 27% faster than the A18 Pro (iPhone 16 Pro Max) on GPU compute, 14.1ms against 19.4ms median. That is well short of Apple's "2x sustained performance" claim.

### Is the A20 Pro faster than the A18 Pro once the phone gets hot?

No. Fully throttled, the A18 Pro in the larger 16 Pro Max was 10–15% faster than the A20 Pro in the 18 Pro (18.7–21.8ms against 21.9–23.7ms median). The cold-start advantage inverts under sustained load. I could not separate chip from chassis, because I tested the smaller 18 Pro against the bigger 16 Pro Max.

### Do the A18 Pro and A20 Pro perform the same when cooled?

Effectively yes. With a Peltier cooler holding thermal state near baseline, the two phones tied on the same workload, and the A20 Pro's own range (12.4–19.1ms) fully overlapped the A18 Pro's (14.7–15.3ms).

### Is the A20 Pro better for on-device AI?

On average, yes. Scoring through Apple's on-device Foundation Model, the 18 Pro's median was 22.4s against 44.0s on the 16 Pro Max, roughly twice as fast, consistent with its doubled Neural Engine (32 cores against 16). The catch is the tail: the 18 Pro spiked to 408.3s while the 16 Pro Max never exceeded 73.3s.

### What is the A20 Pro throttling problem?

Two of the 30 prompts I ran, both asking for long-form prose, caused every spike over 150 seconds on the 18 Pro. The same two prompts produced no spikes on the 16 Pro Max. My hypothesis, unproven, is a larger on-device model tier enabled by the 18 Pro's 12GB of RAM.

### Is it worth upgrading from the iPhone 16 Pro to the iPhone 18 Pro for performance?

For average speed and on-device AI throughput, the gains are real: the full ten-loop run finished roughly an hour faster on the 18 Pro. For sustained heavy load on a warm phone, my results do not show the advantage Apple's marketing implies. This is one phone per generation, so treat it as one data point.

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "Is the A20 Pro faster than the A18 Pro?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Cold, yes: in my tests the A20 Pro (iPhone 18 Pro) was 27% faster than the A18 Pro (iPhone 16 Pro Max) on GPU compute, 14.1ms against 19.4ms median. That is well short of Apple's \"2x sustained performance\" claim."
      }
    },
    {
      "@type": "Question",
      "name": "Is the A20 Pro faster than the A18 Pro once the phone gets hot?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "No. Fully throttled, the A18 Pro in the larger 16 Pro Max was 10–15% faster than the A20 Pro in the 18 Pro (18.7–21.8ms against 21.9–23.7ms median). The cold-start advantage inverts under sustained load. I could not separate chip from chassis, because I tested the smaller 18 Pro against the bigger 16 Pro Max."
      }
    },
    {
      "@type": "Question",
      "name": "Do the A18 Pro and A20 Pro perform the same when cooled?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Effectively yes. With a Peltier cooler holding thermal state near baseline, the two phones tied on the same workload, and the A20 Pro's own range (12.4–19.1ms) fully overlapped the A18 Pro's (14.7–15.3ms)."
      }
    },
    {
      "@type": "Question",
      "name": "Is the A20 Pro better for on-device AI?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "On average, yes. Scoring through Apple's on-device Foundation Model, the 18 Pro's median was 22.4s against 44.0s on the 16 Pro Max, roughly twice as fast, consistent with its doubled Neural Engine (32 cores against 16). The catch is the tail: the 18 Pro spiked to 408.3s while the 16 Pro Max never exceeded 73.3s."
      }
    },
    {
      "@type": "Question",
      "name": "What is the A20 Pro throttling problem?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Two of the 30 prompts I ran, both asking for long-form prose, caused every spike over 150 seconds on the 18 Pro. The same two prompts produced no spikes on the 16 Pro Max. My hypothesis, unproven, is a larger on-device model tier enabled by the 18 Pro's 12GB of RAM."
      }
    },
    {
      "@type": "Question",
      "name": "Is it worth upgrading from the iPhone 16 Pro to the iPhone 18 Pro for performance?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "For average speed and on-device AI throughput, the gains are real: the full ten-loop run finished roughly an hour faster on the 18 Pro. For sustained heavy load on a warm phone, my results do not show the advantage Apple's marketing implies. This is one phone per generation, so treat it as one data point."
      }
    }
  ]
}
</script>
