## What it does

`grilling` is the interview loop that stress-tests a plan, a decision, or an idea before anyone acts on it. It maps the subject as a **design tree**: every decision branches into the decisions that hang off it, and the interview walks the tree dependency by dependency until no assumption stays unstated.

It does not put every open branch to you. Before asking anything, it tests each candidate question for whether it is a **false question**: a choice that looks open but is already settled by evidence, by a decision on record, by a cheap test, or by not mattering yet. It decides those itself and reports each one. Only **genuine questions** reach you, and they reach you once, in a single numbered **end list** after the whole tree is walked and reviewed.

## When to reach for it

Type `/grilling`, or the [agent](https://www.aihero.dev/ai-coding-dictionary/agent) reaches for it on its own when a task fits. It is the only [skill](https://www.aihero.dev/ai-coding-dictionary/skill) in the grilling family that is model-invoked, so you rarely type it. Usually a skill you *did* type runs it for you.

Typing `/grilling` directly gets you the plain interview and nothing else. Where you want something more than that:

| What you have | Reach for |
| --- | --- |
| You aren't working in a working directory | [grill-me](https://aihero.dev/skills-grill-me): the same [session](https://www.aihero.dev/ai-coding-dictionary/session), under a name the agent will never fire by itself |
| You are in a working directory | [grill-with-docs](https://aihero.dev/skills-grill-with-docs): the same session, and it writes `GLOSSARY.md` and ADRs as it goes |
| An effort too big to hold in one session | [wayfinder](https://aihero.dev/skills-wayfinder): it charts a map and runs grilling inside the decision tickets |
| A question that talking cannot settle: how something should look or feel | [prototype](https://aihero.dev/skills-prototype): build the throwaway version, then come back |
| A skill of your own that needs an interview | Invoke `/grilling` from it, rather than writing another interview |

## False questions and who decides

Every candidate question is classified silently, after the agent has read whatever code, documents and prior decisions bear on it. It lands in one of three places:

| What the candidate turns out to be | What happens |
| --- | --- |
| A false question | The agent decides it and reports an **autonomous decision**: the decision, why it was the agent's to make, and the alternatives it rejected. It does not pause for approval, and you can override any of them. |
| A genuine question | It goes to the end list with a recommended answer. Decisions that depend on it go ahead on that recommendation, marked provisional. |
| Unclear whether it is yours or the agent's | It goes to the end list as a **meta-classification question**: should the agent decide this type of question, or always ask you? Your answer can become a permanent amendment to the skill, but only with your explicit approval. |

A false question is one where a single option wins on every count, a recorded decision or rule already fixes the answer, the [environment](https://www.aihero.dev/ai-coding-dictionary/environment) can answer it, a small test can settle it, the choice is immaterial and cheap to change, it belongs to a branch that may never arise, it serves only a speculative future, the options are not really in conflict, or the requirements already determine the mechanism.

Some questions come to you even when they look false:

- an earlier commitment may need reconsidering, because concrete work exposed a conflict or a defect in it
- the step is destructive, hard to reverse, externally visible, costly, security-sensitive, or outside the authority you have granted
- settling the question by evidence would be disproportionately costly or risky, so you choose whether to fund the investigation
- a downstream consequence could decide whether the upstream direction is viable
- a credible future requirement, backed by concrete evidence, would be costly to accommodate later
- the obvious choice adds a dependency, framework, service or overlapping capability to the codebase
- the code shows what it does, but nothing establishes whether that behaviour is intended.

## The end list

Nothing pauses the walk. A genuine question is parked with its recommended answer, and the walk carries on. When it ends, every open question arrives together, numbered, and each item carries:

- the question itself in red, worded so that "yes" accepts the recommended answer
- enough background that a newcomer could follow it without the transcript
- a worked example of what each option would produce, side by side in a table when the options compare along the same lines
- "doing nothing" named as an option, with what it would leave in place
- the recommended answer, and the provisional decisions that flip if you answer otherwise.

The one question that may come earlier is your name, asked once and folded into the conversation, when the skill does not yet know who you are.

## Profiles and delegated answers

The skill keeps one **profile** per person, stored beside the skill. It records the name you want to be called, an **ask-me scope** (what you want asked directly), a **delegate-to-AI scope** (what you want decided on your behalf) and any rulings on domains that straddle the two. Scopes start empty, so everything is asked until you draw a line, and a scope is written only after the agent shows you the exact profile text and you confirm it. If you correct the name it calls you, it treats that as a different person in the room and switches profiles.

A genuine question that falls only inside your delegate-to-AI scope goes to a separate answering [subagent](https://www.aihero.dev/ai-coding-dictionary/subagent) instead of to you. It comes back as a **delegated answer**: the full question, the answer, what the answer rests on, and the profile line that routed it. A delegated answer is tentative until the end review settles it, and your own answer to the same question always supersedes it. A question that trips one of the exceptions above, matches both scopes, or matches neither still comes to you. Where the answering subagent is not available, the agent says so once and answers delegated questions in the same session.

## The end review

When the tree is walked, and before you are asked anything, a reviewer subagent that never saw the conversation checks every decision made. It is told not to favour a decision because it is already made, because it is easier, or because you chose it.

| What the reviewer would change | What happens |
| --- | --- |
| An autonomous decision or a delegated answer | The change is applied and reported, showing the original answer, the reviewer's concern and the revised answer. |
| A decision you made | It is never applied without your approval. It joins the end list with the reviewer's recommendation and evidence. |

You then get a short summary in three groups (decisions the reviewer approved, decisions it revised, and your own decisions it disagreed with), followed by the end list. Answering the list confirms the summary unless you object. If no reviewer subagent is available, the agent says so and asks you to confirm the summary instead.

## What lives here and what lives in the wrappers

This page covers the mechanism. The wrapper skills' pages answer the questions people ask most often.

| Question | Where it is answered |
| --- | --- |
| The tree, false questions, the end list, profiles, delegated answers, the end review | Here |
| How long a session should run, what to do with a question you can't answer by talking, how to avoid nodding along | [grill-me](https://aihero.dev/skills-grill-me) |
| What gets written to `GLOSSARY.md`, what becomes an ADR | [grill-with-docs](https://aihero.dev/skills-grill-with-docs) |

## Common questions

**It decided things without asking me.**
That is the design for false questions. The earlier version put every open branch to you, including ones that already had a sensible default, and users reported spending their answers on questions whose answer was obvious. Each decision the agent takes is reported as an autonomous decision naming why it was the agent's to make and what it rejected, so you can override it. What it must not decide alone is anything on the exception list above, or anything outside your delegate-to-AI scope. If it decided one of those, the run broke the skill, and saying so reopens the decision.

**What happened to rounds?**
They are gone. Earlier versions asked one question at a time, then a round at a time: every question whose prerequisites were settled. Users reported rounds asked while exploration was still running, which the exploration then rewrote. This version walks the whole tree first, decides the false questions, proceeds on recommendations for the genuine ones, runs the end review, and only then asks. Nothing is asked of you before the evidence is in.

**Can I go back to one question at a time?**
Not as a supported setting. The skill asks every open question in one end list and rules out one question per turn. A list whose items each carry their own background and worked examples can be answered item by item, at your own pace.

**The questions don't make sense without scrolling back.**
Each item is meant to stand alone: background a newcomer could follow, a worked example per option, and no reliance on earlier question numbers. A question that leans on a name invented earlier in the session, or on a bare ticket number, has broken that format. Ask the agent to restate it in full.

**It ran out of questions and started building.**
The skill ends by telling you plainly that the grilling session is done, and acting on the plan is a separate instruction from you. Weaker and faster [models](https://www.aihero.dev/ai-coding-dictionary/model) still skip ahead. Users report this most often on lower-effort or non-frontier models, which collapse "interview until shared understanding" into a couple of questions and an outline. If yours does it, the reliable fix is a line in your own `AGENTS.md` or `CLAUDE.md` telling the agent not to implement without permission.

**Can I cap the number of questions?**
No, and a cap is out of scope on purpose. Some plans need three questions and some need fifty, and a fixed ceiling either truncates the hard case or feels arbitrary on the easy one. The intended control is plain language. Tell it to wrap up, or stop and accept the plan where it stands. If the end list is very long, the cause is usually that the scope was too big. Break the work up and grill the pieces.

**I installed `grill-me` on its own and nothing happens.**
`grill-me` is a one-line skill whose whole body is "run a `/grilling` session", so it needs this skill installed too. The same is true of `grill-with-docs`, which additionally needs [domain-modeling](https://aihero.dev/skills-domain-modeling). Installing the whole set avoids the problem. Installing selectively means installing the primitives as well.

**`grill-with-docs` ran, but it never loaded `grilling`.**
This is a real bug, still unfixed, and users report it across [harnesses](https://www.aihero.dev/ai-coding-dictionary/harness) and models. A skill that names another skill does not reliably cause that skill to load, and `grill-with-docs` names two. The sign is a session with no autonomous-decision reports and no end review, where questions arrive without recommendations or worked examples. That means the model is improvising an interview rather than running this one. Asking the agent directly whether it loaded `grilling` and `domain-modeling` usually recovers it.

## It's working if

- Autonomous decisions appear as the walk goes, each naming why it was the agent's call and what it rejected, and the agent keeps moving instead of waiting for approval.
- Nothing is asked of you mid-walk, except your name once if the skill does not know you yet.
- It reads files and runs small checks rather than asking you something it could have found out.
- It never asks you to choose between vague virtues, or about a branch that may never arise.
- Before any question, you get a short summary in three groups, and a reviewer change to a decision you made arrives as a question, not as a done deal.
- The questions arrive together in one numbered list, each in red, with its background, a worked example per option, a "doing nothing" option, and a recommendation you can accept with "yes".
- It ends by saying plainly that the session is done, and does not start building.

## Where it fits

`grilling` is a **primitive**, not a step you schedule: the single source of truth for the interview technique, kept in one place so every skill that needs an interview reaches for it instead of inventing one. [grill-me](https://aihero.dev/skills-grill-me) and [grill-with-docs](https://aihero.dev/skills-grill-with-docs) are its two user-invoked front doors, and `grill-with-docs` is where the main build chain begins, ahead of [to-spec](https://aihero.dev/skills-to-spec). [wayfinder](https://aihero.dev/skills-wayfinder) runs it to resolve decision tickets, [triage](https://aihero.dev/skills-triage) to grill a vague report into a workable one, and [improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) to walk the tree once you have picked a candidate to deepen. When you are unsure which entry point fits, [ask-matt](https://aihero.dev/skills-ask-matt) routes you.
