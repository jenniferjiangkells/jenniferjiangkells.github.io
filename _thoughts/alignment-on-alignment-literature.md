---
title: "Alignment on the alignment literature"
date: 2026-07-14
summary: Getting my terms straight on safety, alignment, and control — and why a product eval and a safety eval are asking fundamentally different questions.
---
This one's a bit of a side quest. Building HealthChain's agentic surface — and, lately, reading too much existentialist philosophy — has left me stuck on a single tension: healthcare is high-stakes *and* complex. High-stakes says *be careful*. Complex says *there's endless room to experiment*, which is also endless room for things to go wrong. Those two instincts pull against each other, and I've been sitting in the gap between them. It's a high anxiety environment, so sometimes I find myself on [LessWrong](https://www.lesswrong.com).

But reading the alignment crowd as a builder, it took a little internal alignment of my own before I could render what I was finding through the right lens. When I hear evals, I think of [Hamel's guide on LLM Evals](https://hamel.dev/blog/posts/evals-faq/) and [benchmarks](https://ai.nejm.org/doi/pdf/10.1056/AIdbp2500144). It didn't help that words like "safety" and "evals" have become buzzwords and are often gestured at vaguely when they can mean very different things, and in my head I used the terms "safety", "alignment", "control" interchangeably.

So before anything else, here's the vocabulary I had to untangle, roughly:

- **Safety**: the broad umbrella — not letting the system cause harm. Unhelpfully, in healthcare I already use "safety" for *domain* safety (clinical correctness, harm prevention, PHI exposure, hitting the wrong payer), so the word shows up pre-loaded before I've even started.
- **Alignment**: does the model actually *want* what we want? Are its goals and values pointed at the thing we intended, and not some proxy of it.
- **Control**: assume it *doesn't*, and build oversight so it can't do damage anyway. Control doesn't ask the model to be trustworthy; it asks whether we could catch and contain it even if it were actively working against us.

What clicked for me was this: in product-based evals, we assume the model is trying to help and trying to *pass* the evaluation; in safety-based evals, we assume the model itself might be misaligned and trying to *defeat* the evaluation. We can say that AI product-based evals are about reliability and reproducibility, whereas AI control is specifically a research agenda that *uses evals as its method*. They use fundamentally different threat models underneath:

| **Aspect**              | product evals                                       | alignment evals                                                         |
| ----------------------- | --------------------------------------------------- | ----------------------------------------------------------------------- |
| **What's being tested** | Model/agent quality and correctness on a task       | Whether a model could covertly subvert oversight or hide capabilities   |
| **Threat model**        | None — assumes model is trying to help              | Assumes model may be misaligned and adversarial                         |
| **Failure mode caught** | Wrong extraction, bad formatting, missed edge cases | Sandbagging, deceptive sabotage, undetected manipulation                |
| **Method**              | Deterministic scoring against ground truth          | Mock deployment scenarios, red-team elicitation, statistical thresholds |

None of this changes what I ship tomorrow. But I can now feel the gap between *is my agent good at its job* and *would I still be okay if it weren't on my side* — and in something this high-stakes and this complex, I'd like to be able to answer both.
