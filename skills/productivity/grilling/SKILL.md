---
name: grilling
description: Grill the user relentlessly about a plan, decision, or idea. Use when the user wants to stress-test their thinking, or uses any 'grill' trigger phrases.
---

Interview the user relentlessly about every aspect of a plan or design until reaching shared understanding. Walk the design tree dependency by dependency, but do not mistake every apparent choice for a decision the user needs to make.

## Active profile

Resolve this once, at invocation, before the first candidate question is classified, then keep applying it for the rest of the session. The naming and identity-check guidance below is continuous, not a one-time gate; but re-resolution mid-session is reactive only, triggered by an identity-check event (step 4), never on a schedule.

1. **Where profiles live.** One file per person under `.claude/skills/grilling/profiles/`, named by slugifying their preferred name: lowercase, collapse runs of non-alphanumeric characters to a single hyphen, strip leading/trailing hyphens ("Barbara" → `barbara.md`). A single pointer file, `.claude/skills/grilling/active-profile.md`, names whichever profile is currently assumed to be in the room; a missing pointer file, or a missing `profiles/` directory, means the same as `active: none`. Read both the same way this skill's own source is reached (native read in a repo checkout, the skill-source read tool elsewhere). Every pointer write in this section is best-effort: if the environment can't write the skill source, apply the resolution for the current session anyway, say once that it couldn't be persisted, and move on.

2. **Resolve the active profile.**
   - **A valid pointer, or exactly one confirmed profile.** If the pointer names a profile file that exists and its Status line reads confirmed, assume that profile. Otherwise, if exactly one file under `profiles/` is confirmed, assume that one and rewrite the pointer to match (best-effort, per step 1). Repairing a stale or missing pointer avoids re-asking next session.
   - **Nothing confirmed anywhere** (no `profiles/` directory, nothing confirmed in it, and no valid pointer). Identity is unknown: bootstrap it (step 5) at a natural point early in the conversation, never as the literal opening line. Grilling is often invoked mid-conversation, so fold the ask into the first acknowledgment rather than opening with it. It is the one question that may come before the end list, because routing needs the profile.
   - **Multiple confirmed profiles and no valid pointer.** Should not arise in practice: the pointer is written every time a bootstrap completes or a switch is confirmed. If it somehow does, don't build a disambiguation flow for it: ask the user once, directly, which profile is theirs, as an ordinary genuine question (format: Asking the user), and write the pointer once they answer.

3. **Weave the resolved name in, never as a greeting.** Work the active profile's preferred name into ordinary prose: inside a question, an acknowledgment, a summary. A fixed floor keeps "occasionally" from decaying into "never": use the name in the first substantive message after the profile resolves (which doubles as the earliest chance for a wrong-profile correction), again when opening a new branch of the design tree or summarizing progress, and in the Finish closing message. Beyond that floor, use judgment: not every message, and never inside the fenced report formats (Autonomous decision, Delegated answer, Fable disagreement), which are audit records, not conversation. Never open with "Hello, <name>": grilling frequently starts mid-conversation, and a cold greeting would announce a session-start ritual that isn't really happening.

4. **A name correction is an identity-check event.** If the user corrects the name they were just addressed by ("That's not my name! I'm Barbara.") or otherwise indicates a different person is present, resolve identity again immediately, before continuing with anything else:
   - Slugify the stated name. If `profiles/<slug>.md` exists and its Status line reads confirmed, switch: update the pointer (best-effort, step 1), acknowledge the switch briefly using that file's own stored Preferred name ("Switching to your profile, Barbara."), and continue immediately under that profile's scopes. Speaking the stored name back is the safety check: if the file belongs to a different person who shares the name, or to a spelling variant that isn't them, the user hears it and can correct again.
   - If they do correct again (the file under that slug isn't theirs), elicit a distinguishing preferred name (e.g. "Barbara L"), which slugifies to its own filename, and bootstrap under it (step 5). Never overwrite or edit the existing person's file.
   - No confirmed file under the slug: bootstrap a new profile under the stated name (step 5).

5. **Bootstrap: name first, scopes later.** A bootstrap establishes who is present; it is not a scope interview, and it never blocks grilling from proceeding. First check whether `profiles/<slug>.md` for the stated name already exists and is confirmed. If so, this is a switch (step 4), not a bootstrap. Otherwise: elicit the preferred name as one ordinary question folded in naturally (format: Asking the user), confirm the exact name back (it becomes the filename slug), then write the new file at `profiles/<slug>.md`: whole file in a single write, Status confirmed with today's date, both scopes and boundary rulings empty. Empty scopes are valid: under "Expertise routing" they match nothing, so every genuine question is asked (the right default for a person who hasn't drawn any lines yet). Scopes and boundary rulings are added later only through "Expertise routing" steps 2–3's confirm-before-write gate, exactly as before this section existed. If the user declines to give a name, proceed without a profile for this session (no name-weaving, everything asked), and don't re-ask. Finish any completed bootstrap by writing the pointer (best-effort, step 1). If the profile file itself can't be written from this environment, use the confirmed name for the current session only and tell the user the profile must be saved from a session with write access to the skill source (the same fallback "Expertise routing" uses).

## Classify before asking

Before presenting a question:

1. Inspect available code, documentation, prior decisions, and other relevant evidence. Find discoverable facts instead of asking the user for them.
2. Identify the candidate question and meaningful alternatives.
3. Test whether it is a false question under the classes below.
4. Check whether an exception makes human input necessary.
5. Route it as an autonomous decision, a genuine question, or a meta-classification question.
6. If it is a genuine question, apply expertise routing (below) before asking it.

Do this classification silently. Never present a false question and then answer it; report the resulting decision instead. Expertise routing is a separate, later filter: it never reclassifies a false question as genuine, it never overrides an exception, and it never applies to a meta-classification question.

## False-question classes

Answer a candidate autonomously when:

1. **Dominated choice.** One option is superior in every material respect and no real tradeoff remains.
2. **Governing decision.** Existing goals, principles, definitions, rules, guidelines, or recorded decisions unambiguously determine the answer.
3. **Discoverable fact.** Codebase inspection or other available evidence can establish the answer.
4. **Empirically resolvable.** A proportionate test, measurement, or small prototype can settle the issue without asking the user to speculate.
5. **Immaterial and reversible.** The choice has no meaningful effect on behaviour, architecture, risk, or future options and is cheap to change.
6. **Premature branch.** The question belongs to a downstream branch that is not needed to resolve the current upstream decision and may never arise. A cost/coverage tradeoff among mechanisms where the recommended option is to defer and track (committing no resources and foreclosing no alternative) is a premature branch, not exception 3.
7. **Unsupported future speculation.** The question exists only to accommodate a hypothetical future requirement without concrete evidence.
8. **Abstract non-tradeoff.** The question asks the user to choose between vague virtues without identifying a concrete point of conflict.
9. **False dichotomy.** The alternatives are compatible, complementary, or otherwise not mutually exclusive.
10. **Means determined by outcomes.** Known requirements and constraints determine the implementation mechanism; the user need not select the mechanism itself.

For every autonomous decision, report:

```text
Autonomous decision: <the decision>
Why autonomous: <false-question class>. <case-specific explanation>
Decided against: <meaningful rejected alternatives and why>
```

Include only meaningful rejected alternatives; never invent alternatives to fill the format. Do not ask for approval or pause. Continue immediately to the next candidate question, and keep going until the design tree is complete. The user may override any reported autonomous decision.

## Exceptions that require the user

A candidate that resembles a false question still requires human input when:

1. **A governing commitment may need reconsideration.** Implementation or concrete design work exposes unforeseen implications, conflicts among commitments, unreasonable costs, or a defect in an earlier commitment.
2. **Consequential authorization is required.** The action is destructive, difficult to reverse, externally visible, costly, security-sensitive, or outside authority already granted. The confirmation authorizes the consequence rather than re-deciding the reasoning.
3. **Decisive evidence is disproportionately costly or risky.** The user must choose whether to fund the investigation or which uncertainty, delay, cost, or risk to accept. Do not ask the user to guess the empirical result.
4. **Downstream implications decide the upstream direction.** A downstream consequence must be understood because it could materially change whether the upstream branch is viable. Explore only the consequences needed for that decision.
5. **A credible future requirement creates lock-in.** Concrete evidence such as a committed roadmap, contractual obligation, or known regulatory change makes the requirement credible, and accommodating it later would be costly or impossible.
6. **The locally obvious choice expands the technology surface.** The choice adds a dependency, runtime, framework, service, build tool, or materially overlapping capability, creating repository-wide ownership and maintenance costs.
7. **Current behaviour does not establish intended behaviour.** Inspection shows what the software does, but available tests, documentation, and conventions do not establish whether that behaviour is deliberate, accidental, or obsolete.

A genuine question or an exception does not stop the walk. Defer it to the end list (Asking the user) with its recommended answer. Where later decisions depend on it, assume the recommended answer, mark each dependent decision provisional, and continue.

(How to present the evidence behind each question in that list is `/probanda:show-me`: rendering evidence rather than describing it, showing full context rather than a curated slice, and verifying any missing-data claim firsthand before stating it. Its one-decision-per-turn rule does not apply here: the end list follows Asking the user.)

## Asking the user

This section is the one home for how a question is put to the user. Other files point here rather than restate it.

**When.** With several open questions, ask them in one numbered list at the end of the turn (the end list), never one per turn or one per message. The bootstrap name question (Active profile, step 2) is the only one asked before it.

**What each item carries.**

- Full background, so a newcomer to the project could follow it without the transcript: what the question is about, why it arose, and what depends on it.
- A worked example for each option: what that option would concretely produce. When the options compare along the same lines, show them side by side in a table.
- "Doing nothing" named among the options, with what it would leave in place.
- Recommendations marked by the options rule in the global CLAUDE.md (Working rules, Questions).
- The recommended answer, and the provisional decisions that flip if the question is answered the other way.

**How it looks.** Every question the user must answer is shown the same way, wherever it reaches them: the question in red, the explanation beneath it in normal text. Red marks what only the user can settle, so nothing else is red: not an autonomous decision, a delegated answer, a reviewer's disagreement, or a summary line.

Write the question as inline math: `$\color{red}{\text{Should the duration cap be 30 minutes?}}$`. In a list, an item looks like this:

```markdown
1. $\color{red}{\text{Should the duration cap be 30 minutes?}}$
   Background in normal text. Options, each with a worked example: 30 minutes (recommended: beats doing nothing because ...), 60 minutes, doing nothing (no cap). **Recommended: 30 minutes.** The provisional decisions that flip if it is answered the other way.
```

Word each question that carries a recommended answer so "yes" accepts that answer. Agreeing then never means answering "no". The meta-classification question keeps its fixed wording.

A question folded into prose, such as the bootstrap name question, is the same red span without a list number. Math does not line-wrap, and the app shows any `$…$` span longer than 60 characters as raw code. So keep the text inside `\text{}` to 40 characters or fewer, counting each backslash, and put a longer question on several lines, one span each. Inside `\text{}`, write no backslash and none of `%`, `$`, `&`, `#`, `_`, `{` or `}`. When a question needs one, reword it (`#1262` becomes `issue 1262`), or move that text into the normal-text explanation beneath the question. Write no straight double quote (`"`) inside `\text{}`. When a question needs a quoted phrase, reword it so the span needs no quotes, or move the quoted phrase into the normal-text explanation beneath the question.

## Meta-classification and self-improvement

Use this only after investigation and careful application of the classes and exceptions. When they still do not unambiguously determine whether the candidate belongs to the model or the user, defer it to the end list worded exactly:

> Is this the type of question I should decide autonomously, or the type of question that I should always ask you?

List the candidate question and its recommended answer directly beneath it, so one answer settles both. In the list it is red like any other question (Asking the user), as two spans split after the comma. Once it is answered, put the proposed amendment to the false-question classes or exceptions that would classify similar cases in the future, and the request for approval to edit this skill, in the next list (see Finish, "After the answers"), never as a turn of its own. Do not edit the skill before explicit approval.

If approved:

1. Locate the authoritative `grilling` skill source that supplied these instructions. Do not edit wrappers or duplicate the taxonomy elsewhere.
2. Make only the approved amendment and verify the saved text.
3. Report the change, then re-derive the candidate and its dependents under the new rule.

If declined, apply the user's classification for the current session without changing the skill, then resume. If the authoritative source cannot be identified or written, report that failure rather than pretending the skill improved.

Approved self-improvement is a narrow exception to the rule against enacting the plan under discussion. It authorizes only the agreed amendment to this skill.

## Expertise routing

A second filter, applied only to a candidate that survives classification above as a genuine question. It never touches an autonomous decision, never overrides an exception, and never applies to a meta-classification question (delegating "should I ask you?" would be circular). A question that trips an exception (above) always goes to the user, regardless of what the profile says.

Before asking, read the active profile at `.claude/skills/grilling/profiles/<slug>.md`, reached the same way this skill's own source was reached (native read in a repo checkout, the skill-source read tool elsewhere). It is resolved once at invocation and re-resolved on any identity-check event (see "Active profile," above). It records, in the user's own words: the name they want to be called (elicited and confirmed once, during that profile's bootstrap, and not re-asked here), the **ask-me scope** (domains and question-types they want asked directly), the **delegate-to-AI scope** (domains and question-types they want the model to decide on their behalf), and any **boundary rulings** on domains that straddle the two. A confirmed profile with empty scopes is valid: it matches nothing, so everything is asked.

**Treat the active profile as absent unless its Status line reads confirmed with a date.** A file that is missing, internally inconsistent, or apparently half-written is an absent profile: route nothing on it, tell the user what you found, and re-elicit before trusting it again. Before any profile is confirmed (including mid-bootstrap, before the name is confirmed), there is no active profile at all: treat that the same way, and default to asking.

1. **Match the candidate to a scope.** Check boundary rulings first: a recorded ruling on this straddle resolves the match.
   - Falls only in the ask-me scope → defer it to the end list with a recommended answer.
   - Falls only in the delegate-to-AI scope → route it as a delegated answer instead of asking (below).
   - Anything else (matches both scopes, matches neither, the match is unclear, or there is no confirmed profile) → defer it to the end list. A question can genuinely match both scopes at once (a library choice is an implementation mechanism *and* a product commitment); both-scope and unclear matches always default to asking, never to delegating.
2. **Only write or update the active profile's file when the user's own account draws a line precise enough that the two scopes don't overlap and don't leave the domain that prompted this decision ambiguous.** Partial coverage is fine and doesn't need to mention every domain: "ask me about UI, don't ask me about anything technical" is complete on its own. If the account is ambiguous, or could place the same question type in both scopes, put clarifying questions in the list rather than saving anything. If the user explicitly says to stop clarifying, stop: save nothing, change nothing about their profile, and treat that as a resolved, valid outcome, not a failure. When the user resolves a both-scope question, offer to record the resolution as a boundary ruling, under this same gate.
3. **Before writing or updating, show the user the exact resulting profile text (both scopes and any boundary rulings) and get explicit confirmation (the confirmation question in the Asking the user format, the profile text beneath it). Only write after that confirmation, and write the whole file in a single write** (never a partial edit), setting the Status line to confirmed with today's date. A profile records current preferences only: scopes and rulings as standing rules, with no dates, provenance, or superseded entries; when a preference changes, rewrite the affected line rather than appending history. If the active profile's file cannot be written from this environment (a read-only skill-source view, for example), apply the confirmed scopes for the current session only and tell the user the profile must be saved from a session with write access to the skill source, mirroring how skill self-improvement handles an unwritable source.

**Where a delegated answer's authority comes from.** A delegated answer is not the agent standing in for the user. The delegate-to-AI scope is the user's own standing instruction, confirmed in their own words, and it authorizes only a *tentative* answer. Nothing is settled by the agent's answer alone: a delegated answer becomes a decision only through the end review at Finish, or through the user answering it themselves. Anyone auditing the transcript must be able to trace every delegated answer to the profile line that authorized it and to the review that settled it.

**Who answers a delegated question.** Not the interviewing session. Send it to a `grilling-answerer` subagent: spawn one the first time a question is delegated, then message that same agent for each later one, so its answers build on its earlier ones. Run a second only when the design tree has two branches that do not depend on each other. Each message carries the question in full and the context needed to answer it, because the subagent sees no transcript. Send a small batch in one message when several delegated questions are independent. If it returns `Answer: belongs to the user`, the question is a genuine question: defer it to the end list. If the agent type is not available in this environment, say so once and answer delegated questions in-session.

When a question routes to the delegate-to-AI scope, never skip it silently. Report it as one self-contained block, the full question included, with the subagent's answer and basis, so the user always sees what was delegated and what the answer rests on:

```text
Delegated answer (recommendation): <the question, in full>
Answer: <the model's answer>
Basis: <what was consulted: files, tool results, prior decisions>
Routed by: <the delegate-to-AI scope line or boundary ruling>
```

The grade in the header follows `shared/answer-grades.md` (read with `mcp__plugin_probanda_data__skill_source__read`), read the same way this skill's own source is reached: hunch (nothing in the session consulted), recommendation (formed from what was already in reach), advice (a recommendation plus an independent reviewer's own evidence). The default grade for a delegated answer is **recommendation**, because step 1 of "Classify before asking" has already inspected the code, documentation and prior decisions before the question was routed. Write `(hunch)` only when that inspection turned up nothing on the point and the answer rests on training knowledge alone, and say so in `Basis:`. A delegated hunch should be rare.

Do not wait after a delegated answer. Continue immediately to the next candidate. Never use the "Autonomous decision" format for a delegated answer, and never assert one as settled: whatever its grade, it stays tentative until the end review at Finish resolves it (below). Where a later question depends on a delegated answer, proceed on that answer and mark what is built on it provisional. The user may answer a delegated question at any time; if they do, their answer supersedes the delegated answer entirely and the exchange is treated as a normal genuine-question answer.

## End review

When the design tree is walked, review before asking the user anything.

1. **Spawn one `reviewer` subagent, in the foreground, for a decision review.** It sees no transcript, so its brief carries: every decision made this session with its basis (each Autonomous decision and Delegated answer block verbatim, and each question the user answered with their answer), each deferred question with its recommended answer, what is marked provisional and what it depends on, and the plan or design under discussion. Tell it to check every decision for errors the user or a model may have made, and to reconsider each one. Tell it not to favour a decision because it is already made, because it is easier, or because the user chose it: the goal is the best decision.
2. **A change to a model's decision is applied.** That covers autonomous decisions and delegated answers. Report each as:

   ```text
   Fable disagreement: <the original question>
   Original (recommendation): <the original answer>
   Fable's concern: <what the reviewer flagged>
   Revised (advice): <the corrected answer>
   Basis: <original basis + the reviewer's evidence>
   ```

   Write the header grades as they actually are: `Original (autonomous)` for an autonomous decision, `Original (hunch)` for a hunch; `Revised (advice)` when the reviewer brought evidence of its own (a constraint, a counter-example, a file it read), `Revised (recommendation)` when it only questioned the reasoning and the fix came from evidence already in hand. Agreement alone does not change a grade: an item becomes advice only when the reviewer's agreement brings its own evidence.
3. **A change to a decision the user made is never applied without the user's express approval.** Each one joins the end list: their decision, the reviewer's recommendation, and the reviewer's evidence. The same goes for a decision the reviewer says belonged to the user.
4. **Re-derive what changed.** If a revised or re-decided answer has dependents, work them through again and send only those to the same reviewer once more. A new question that surfaces is routed like any other.
5. If no reviewer subagent is available in this environment, say so, and fall back to asking the user to confirm the summary (format: Asking the user).

## Finish

Keep track of every resolved item: user-answered decisions, reported autonomous decisions, and delegated answers. After the end review, give a brief summary, in this order:

1. **Decisions made and approved by the reviewer.** Group delegated answers by grade: advice (name the reviewer's evidence), recommendation (reviewed, no issue raised), and hunch (rare, and always listed as provisional until the user answers it or the point is checked against evidence).
2. **Decisions the reviewer revised on reconsideration**, each with what it was and what it became.
3. **The user's own decisions that the reviewer disagreed with**, each with the reviewer's advice, pending in the end list below.

Say "none" for an empty list. Do not ask the user to confirm the summary. They may override any item at any time.

**End the turn with the end list.** After the summary, put every deferred question and every item the end review sent to the user in the end list, each item as Asking the user sets out. Answering the list confirms the summary unless the user objects. If nothing is left to ask, tell the user plainly that the grilling session is done.

**After the answers.** If an answer differs from the recommended one, or approves a change to the user's own decision, re-derive the affected dependents, send only those to the same reviewer once more, and report what changed. A new question that surfaces goes into a new list the same way. Once no question is left, tell the user plainly that the grilling session is done. Enacting the plan under discussion is a separate instruction from the user.
