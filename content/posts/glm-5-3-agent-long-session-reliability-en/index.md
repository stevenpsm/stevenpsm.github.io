---
title: "After a 60+ Turn Agent Session: Six Guardrails I Added to GLM-5.3"
date: 2026-09-28T12:00:00+08:00
draft: false
tags: ["English", "LLM Agent", "GLM-5.3", "pi", "AI Coding", "Reliability Engineering"]
categories: ["Tools and Practice"]
description: "Six recurring reliability failures I observed during a long GLM-5.3 and pi coding agent session, and how I turned them into practical, testable guardrails."
lang: "en-US"
alternateLang: "zh-CN"
alternateURL: "/posts/glm-5-3-agent-long-session-reliability/"
xDefaultURL: "/posts/glm-5-3-agent-long-session-reliability/"
ShowToc: true
TocOpen: false
---

> This article is available in two languages: **English** · [中文](/posts/glm-5-3-agent-long-session-reliability/)

I recently used GLM-5.3 through the pi coding agent for a session that ran for more than 60 turns. It was not a single coding task. The work moved continuously through requirements, code changes, tests, background jobs, deployment, and progress reporting.

The experience left me with mixed feelings.

On one hand, GLM-5.3 can clearly handle complex code and long chains of work. Its official materials also present complex coding and long-horizon agentic tasks as important goals for the model. On the other hand, the longer the session ran, the more I noticed problems that were not really about whether it could write a particular piece of code. It reported progress selectively. It was drawn toward newer and more interesting tasks. It declared features missing before checking. It sometimes blamed the environment for gaps in its own implementation.

None of these problems looks especially dramatic in isolation. The trouble is that they reinforce one another over a long task. An unverified claim triggers duplicate implementation. The duplicate implementation introduces a regression. The regression is then obscured by optimistic reporting. Meanwhile, a small task that the user already approved can remain untouched for many turns as new ideas keep arriving.

I did not want to end with the vague conclusion that “the model is unreliable sometimes.” Instead, I organized what happened into six recurring patterns and tried to compress the mitigations into rules that could be placed directly in an agent's runtime context.

## First, this is not a model benchmark

The [official GLM-5.3 project](https://github.com/zai-org/GLM-5) describes the model as being stronger at complex coding and long-horizon tasks. This article is not an attempt to dispute those benchmarks, nor is it a ranking of model intelligence.

My sample is one real but uncontrolled long session. Its outcome was shaped by the model version, reasoning budget, context compression, pi's agent loop, tool permissions, project complexity, and the way I prompted it. Observing a behavior does not prove that the model alone caused it, much less that every LLM behaves the same way.

So when I use the word “pattern” below, I only mean that similar behavior appeared repeatedly at different points in this session. The root causes remain hypotheses. Whether the mitigations work needs to be tested with controlled comparisons.

That is also why I eventually created a separate knowledge base: to keep field observations, general hypotheses, runtime instructions, and validation results apart, instead of letting experience harden too quickly into untested doctrine.

## Six problems that kept recurring

### 1. Selective reporting: telling me how much was done, but not how much existed

An agent can produce very reassuring progress reports: “46 completed,” “all tests passed,” or “coverage increased from 0 to 23.” The numbers may be true. The problem is the missing denominator and the missing boundary around the conclusion.

“46 out of 83” feels different from “46 completed.” A green test suite does not necessarily mean the user's system-level objective has been achieved. A large increase can still leave absolute coverage very low.

I ended up adding a simple reporting contract: bad news before good news, every completion number must include its denominator, and command success, test success, feature implementation, and system-goal completion must remain distinct. If the denominator is unknown, the agent must say “denominator unknown” rather than quietly omit it.

### 2. Novelty-driven task switching: a new architecture is always more attractive than filling in parameters

When several tasks are available, the agent showed a clear preference for exploratory work. Building a new framework, designing a pipeline, or integrating a new tool could hold its attention. Filling in parameters, updating documentation, and fixing small bugs repeatedly slipped to the back of the queue.

The clearest cases happened after the user had already said, “Yes, do it.” The agent acknowledged the approval, but the next complex problem pulled it away again.

The remedy is not another reminder to “be diligent.” It is an explicit queue: `approved / active / blocked / done`. A user-approved item goes to the top immediately, and unrelated work does not begin until that item is complete or genuinely blocked. A work-in-progress limit also helps prevent the agent from manufacturing a sense of progress by continuously opening new threads.

### 3. Unverified assertions: turning “I did not see it” into “it does not exist”

Several times during the session, the agent concluded that a feature had not been implemented. In reality, a complete implementation existed in a standalone script; it simply had not been connected to the current entry point. Components discussed early in the session were later proposed again as if they needed to be built from scratch.

The important correction was to separate five states: absent, implemented but not integrated, integrated but disabled, enabled but unverified, and verified. A single word such as “missing” cannot stand in for all five.

I turned “check before answering” into a hard rule. Any claim about the existence of a feature or the current state of a system must be preceded by a search, configuration inspection, or read-only probe. When the user says, “I think that already exists,” the first response should be to verify it again, not to protect the consistency of the previous answer.

### 4. Blind waiting and promises about the future: waiting is not monitoring

When a background task was running, the agent sometimes chose a long `sleep`. If the task failed in its first second, the rest of the wait was wasted. If the task had stalled, checking only whether the process still existed would not reveal the problem.

There was also another kind of failure: promising, “I will check again in a few minutes.” Whether that promise is possible depends on the agent harness. If the harness has no timer or autonomous wake-up mechanism, the agent cannot return on its own. It must wait for the next user message.

A more reliable approach is to check immediately after starting a task, prefer waiting on an explicit completion condition, and use short intervals, a hard limit, and a cancellation path when polling is unavoidable. Each check should cover errors, liveness, and progress delta. Two consecutive checks with no progress should change the mode from “keep waiting” to “diagnose a possible stall.”

### 5. Attribution bias: blaming the environment before checking its own code

When an operation fails, explanations such as “the environment does not support it,” “the tool is incompatible,” or “the system was designed this way” are convenient. Some failures in this session eventually turned out to be a missing adapter handler, an incorrectly formatted parameter, or an existing implementation that had never been connected to the active entry point.

I imposed a fixed order for failure attribution: inspect implementation defects first, integration gaps second, and external environmental limits last. The third layer cannot become the conclusion until the first two have been ruled out with evidence. When the cause is still unknown, “not yet localized” is more honest than selecting an explanation that merely sounds plausible.

### 6. Fixes that introduce regressions: the target test turns green while the overall metric falls

When fixing a bug, an agent can easily focus only on the case that exposed it. Replacing a shared handler may fix the original input while breaking another consumer. The damage might remain hidden until a later full run shows that the overall metric is worse than before the fix.

The mitigations include listing consumers before changing a shared component, preserving a pre-change baseline, running the target regression first, and then choosing a broader test set in proportion to the impact. A high-risk shared component deserves a full suite. A small local change can use a narrower regression set if the reason is recorded.

I deliberately did not write “always run every test after every change” as a universal rule. That turns reliability into an expensive ritual that people and agents eventually route around. The more practical principle is that test scope must match risk, and every newly introduced failure must be explained.

## Why not simply rename the long document `AGENTS.md`?

The [pi coding agent](https://github.com/earendil-works/pi) can load an `AGENTS.md` file from a global or project directory. My first instinct was to put all six problems and the complete analysis into that file.

After another look, I no longer thought that was a good structure.

A research document needs to preserve context, root-cause hypotheses, counterexamples, and evidence limits. Runtime context needs short, explicit, checkable actions. Combining the two wastes context and makes the instructions that matter hardest to find.

So I created a new repository: [Agent Reliability Patterns](https://github.com/stevenpsm/agent-reliability-patterns). It is not tied to one model. The material is divided into five layers:

```text
cases/       Sanitized field cases and evidence
patterns/    Full model and harness observations
guardrails/  Short rules for system prompts or AGENTS.md
evals/       Repeatable comparative evaluations
templates/   Templates for later reproductions and experiments
```

The first observation document is named:

```text
patterns/glm-5.3-pi-long-session-failure-modes.md
```

The material meant to be copied into runtime context lives here:

```text
guardrails/long-session-reliability-guardrails.en.md
```

This split has another benefit. If a pattern fails to reproduce in another model, I do not need to rewrite a supposedly universal `AGENTS.md`. I can update the pattern's evidence status instead.

## Guardrails have to do more than sound reasonable

There is a familiar trap in prompt engineering: write a set of rules that sounds unquestionably correct, then assume the problem has been solved. A model's ability to repeat “check before answering” does not mean it will actually check next time.

I plan to track more direct metrics for these guardrails:

| Metric | What it is meant to answer |
| --- | --- |
| Denominator completeness | When the agent reports completed work, how often does it also report the total? |
| Debt visibility | How many known failures appear in the main summary rather than an appendix? |
| Approved-task age | How many turns pass between user approval and actual execution? |
| Verification-first rate | How many existence claims include search or probe evidence? |
| Zero-progress detection delay | How long does a background task remain stalled before the agent notices? |
| Attribution evidence rate | Did a failure conclusion rule out implementation and integration problems first? |
| Regression escape rate | How many failures introduced by a fix were not found before commit? |

Future validation needs to hold the model version, reasoning budget, harness version, context strategy, tool permissions, and task input steady before comparing runs with and without guardrails. Otherwise, an apparent improvement may be sampling variance, while an apparent regression may come from context compression or a changed tool environment.

## What I want to collect is not a list of model complaints

This work began with my experience of GLM-5.3, but I do not want the repository to become a catalog of flaws in different models.

The more useful questions are: Which failures reproduce reliably? Which belong to a particular harness? Which can be reduced by a context contract? Which require an external state machine, test system, or product mechanism?

Some constraints belong in a prompt, such as requiring a denominator in every progress report. Some belong in code, such as generating coverage metrics and regression diffs automatically. Others must come from the harness, including scheduled wake-ups, persistent task queues, and background-task events. Not every engineering responsibility can be pushed back into model context.

One session of more than 60 turns is not enough to produce universal answers. It was enough to make me start building a more testable way to record what happens. Instead of hoping that an agent will always remember to “be more reliable,” I would rather give each reliability requirement a trigger, an observable signal, and a failure gate.

That may be the clearest thing I took away from using a long-horizon agent: model capability determines how far it might go; task state, validation mechanisms, and engineering guardrails determine whether it still remembers why it started once it gets there.
