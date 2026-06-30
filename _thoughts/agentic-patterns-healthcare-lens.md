---
title: "Agentic design patterns, read through a healthcare AI lens"
date: 2026-06-30
summary: Mapping Anthropic's agent patterns onto healthcare — and stumbling into why verifiability is the question that matters.
---

I read Anthropic's guide on [Building Effective AI Agents](https://www.anthropic.com/engineering/building-effective-agents) to re-familiarize myself with common agentic engineering patterns. The whole thing was simple, concise, and a pleasure to read. I find that the most elegant technical writing often offers the simplest advice.

I went in planning to do something mechanical: take each pattern in the guide and find a healthcare use case for it. What I didn't expect was for the exercise to flip on me partway through. Somewhere around the third pattern, the patterns stopped being the interesting part, and a different question quietly took over: _which problems in healthcare are actually verifiable?_ That turned out to be the thread worth pulling, and it's where this post ends up. But the order I found it in is half the point, so let me walk through it the way it actually happened.

My takeaway for the first principles of agentic systems: **simple > complex**, **transparency > abstraction**. It's important to remember that agents are just LLMs with tools, memory, and retrieval, set loose to interact with an environment. The craft is in the restraint of tailoring to a use case without overcomplicating the design, and patience to document and explain your tools clearly. It's very endearing to me to think of LLMs as junior devs.

So here's where I think each pattern could land through a healthcare AI lens. (As someone who's deployed healthcare AI into the real world, I know this raises a mountain of infrastructure and privacy questions — but here I'm imagining purely greenfield.)

## Workflows

### Prompt chaining
- **Generating clinical documents from speech.** Transcribe a consultation, then chain steps to shape the raw text into a note that follows a fixed structure like SOAP — each step doing one job (transcribe → structure → validate).
- **Translating clinical-trial criteria into plain language.** A staged translation from dense medical jargon to a readable, layperson-friendly summary, each link in the chain stripping away a layer of complexity.

### Routing
- **Medical Q&A triage.** Send logistical questions ("when's my appointment?") to a data-fetch tool, general health queries to an LLM, and anything clinically complex or ambiguous to a human. Here the routing _is_ the safety mechanism.

The next few patterns I struggled to find use cases for — they all looked like routing with extra steps, and the guide's own examples got more abstract as it went. Then it clicked: maybe the point isn't to slot one use case into each workflow, but to see them as a graduation from simple → complex that grows with your requirements. That's hard to pin down in a high-stakes domain like healthcare, where there's _always_ one more guardrail you could add. Which raises the real question: when is it good _enough_? Enter the world of evals. (More on that later.)

Many of the guide's examples also lean coding-related. Unsurprising, given the explosion of AI-assisted dev tools, and given how Dario Amodei framed it at this year's [Code with Claude](https://www.youtube.com/watch?v=ZC-XCr106T0): coding was the first beachhead because it was _verifiable_, and the next frontier is making more domains verifiable.

This made me reconsider [FHIR](https://hl7.org/fhir/) in a new light. Maybe it doesn't have to be a pain in the ass and a means to an end. FHIR is standardized, structured JSON. That makes it verifiable — and that makes _certain problems in healthcare verifiable too._

So then I redirected the question: which agentic use cases in healthcare are actually _verifiable_?

### Parallelization
- **Converting free text to FHIR resources.** Run parallel sub-calls that each validate a different facet of the generated FHIR JSON — structure, codes, encoded values — before anything is returned. The structure gives you something concrete to check against.

### Orchestrator–workers
- **Aggregating health records across sources.** An orchestrator fans out to workers that each pull and normalize records from a different system, then reconciles them into one coherent picture.

### Evaluator–optimizers

I spent a good chunk of time thinking about this one. The evaluator is most useful when:

a) there's a clear quality bar *and* iterating against it measurably improves the output, and

b) the feedback is something an LLM can give on its own, without human supervision.

The cleanest example is de-identification. A generator produces a redacted version of a clinical note; an evaluator scans it for any residual PHI — a stray name, an MRN, a date sitting where it shouldn't — and hands back whatever leaked; the generator redacts again, and the loop repeats until the pass comes back clean. The bar is almost binary (is there still PHI, yes or no?), the feedback is something an LLM can give itself, and each round is verifiably better than the last: a loop you can close and run all day, given clearly defined success criteria.

Clinical coding has the same shape: generate ICD codes from a note, evaluate whether each is supported by the text and specific enough, revise. The moment the feedback requires *clinical* judgment, though ("is this safe", "is this the right call"?), the loop has to open back up to a human.

This points to something I keep circling back to: healthcare AI isn't a monolith where everything is uniformly high-risk. There's a spectrum, and the evaluator–optimizer pattern works precisely on the low-risk, self-verifiable end of it.

## Agents

Two strands here, and they sit at opposite ends of that same spectrum.

Agents that make *clinical decisions* can never run without a human in the loop, full stop. The action space is open-ended, the stakes are someone's health, and "verifiable" stops being something you can fully automate.

But agents that reconcile healthcare *data* can, I think, be fully agentic — especially when they are in a standardized, structured format like FHIR. This is a step beyond the orchestrator–workers example above, and the difference is autonomy. Workflows are *bounded*: fan out, pull, normalize, synthesize, done. A full agent is for when reconciliation is *open-ended*: it decides what to fetch next, resolves conflicts between sources as it finds them, and keeps looping against the data until it's coherent, with no predetermined number of steps.

The key is that the output is still structured and checkable, which is why that autonomy is safe here: the more verifiable the task, the more autonomy an agent can hold.

The real limitation is the test environment to run it in. Do we have rigorous enough sandboxes to fully stress healthcare-data fidelity, to experiment in a greenfield direction, *before* any of this touches real PHI? That's easy to imagine and hard to earn. *C'est la vie*!

---

So where does that leave us? Less of a checklist of patterns to force healthcare into, and more of a gradient of autonomy you *earn* through verifiability. The parts of healthcare that are structured — FHIR, data reconciliation — are the parts you can verify, and the parts you can verify are the parts an agent can own. The clinical-judgment end stays human, not because agents can't reach it, but because we can't yet *measure* them well enough to trust them there.

And now we loop full circle back to the question I parked earlier: when is it good *enough*? Verifiability is what makes that question answerable at all: you can't measure "good enough" in a domain you can't check. Time to check out the world of evals!
