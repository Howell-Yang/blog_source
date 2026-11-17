---
title: Urban NOA on a new chip (AXERA M95)
summary: "Technical lead for porting QCraft's urban navigate-on-autopilot stack to a new chip family: a staged bring-up from a 700-TOPS demo board to the 350-TOPS M95, model adaptation and quantization, and road-test issue clustering. The v4 stack runs on test vehicles."
org: QCraft
period: 2026
role: Technical lead (since June 2026)
weight: 6
metrics:
  - value: "350 TOPS"
    label: target chip, J6M class
  - value: "v4"
    label: stack running on test vehicles
  - value: "0"
    label: frame drops on key models in road tests
topics: [AXERA M95 / M97, Urban NOA, Bring-up, Quantization, Program leadership]
---

## Why this chip, and why early

Our stack ran on Horizon chips. Once Horizon started selling its own driving stack, depending on one chip vendor became a commercial risk. AXERA's M-series offered competitive compute per cost: the M95 (350 TOPS) is comparable to Horizon's J6M class, and few chips above 100 TOPS were available at the time.

Experience with J6M and Qualcomm showed that **the value of an adaptation falls month by month**. The first teams with a working stack on a new chip get the customer conversations. So the program started before the M95 was ready, on an M97 (700 TOPS) demo board.

## Plan

{{< pipeline "M97 demo board|prove the stack" "Model adaptation|operators, quantization" "M95 bring-up|timing + accuracy" "Vehicle tests|v4 stack" "Issue clustering|feed back to owners" >}}

1. **Prove the stack on the demo board, timing second.** On the 700-TOPS board, the priority was correct behavior: driving first, then parking. Latency could wait.
2. **Adapt the models.** Adapt and quantize each model for the new toolchain, and check numerical consistency with the same [stage-by-stage method](/projects/numerical-consistency/).
3. **Bring up on the M95** and tune latency until the driving models fit the frame budget with headroom.
4. **Test on vehicles** with the v4 stack, and run development test analysis.
5. **Cluster road-test issues** so each goes to the right owner with evidence attached, instead of a list of anecdotes.

## Results

{{< callout "result" >}}
The v4 urban NOA stack runs on test vehicles on the M95. Key models showed no dropped frames in road tests, and latency work continues to add headroom.
{{< /callout >}}

Road-test findings were grouped into themes, each with its owner:

- **Lane-change safety:** some lane changes came too close to vehicles alongside, or were taken despite traffic approaching from behind.
- **Speed behavior:** too conservative on some open roads, and slow to accelerate after yielding to vulnerable road users.
- **Navigation lane choice:** wrong lane selection in some scenarios.

These are mostly model-behavior issues, which the model teams continue to improve. Separating "the port is wrong" from "the model needs work" keeps a bring-up moving.

Merging the chip-specific code back into the main branch also exposed integration bugs, which were fixed so the port did not become a long-lived fork.

## What this taught me

- **Sequence by value.** Correct behavior on a forgiving board first, then performance on the target.
- **Reuse the method, not just the code.** The consistency checks built for Horizon chips transferred directly.
- **Triage is leadership work.** Clustering issues with evidence and owners kept a multi-team effort moving.
