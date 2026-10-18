---
name: wayfinder
description: Plan a huge chunk of work (more than one agent session can hold) as a shared map of decision tickets on your issue tracker, and resolve them one at a time until the way to the destination is clear.
disable-model-invocation: true
---

A loose idea has arrived, too big for one agent session, and wrapped in fog: the way from here to the **destination** isn't visible yet. Wayfinding is about finding that way, not charging at the destination. This skill charts the way as a **shared map** on the repo's issue tracker, then works its **decision tickets** (questions whose resolution is a decision, not slices of a build to execute) one at a time until the route is clear.

The destination varies per effort, and naming it is the first act of charting: it shapes every ticket. It might be a spec to hand off and iterate on, a decision to lock before planning starts, or a change made in place like a data-structure migration. The map is domain-agnostic: engineering work, course content, whatever fits the shape.

## Plan, don't do

Wayfinder is **planning** by default: each ticket resolves a decision, and the map is done when the way is clear, with nothing left to decide before someone goes and does the thing. The pull to just do the work is usually the signal you've reached the edge of the map and it's time to hand off. An effort can override this in its **Notes**, carrying execution into the map itself, but absent that, produce decisions, not deliverables.

## Refer by name

Every map and ticket is an issue, so it has a **name**: its title. In everything the human reads (narration, the map's Decisions-so-far), refer to it by that name, never by a bare id, number, or slug. A wall of `#42, #43, #44` is illegible; names read at a glance. The id and URL don't vanish; a name wraps its link, but they ride _inside_ the name, never stand in for it.

## The Map

**Two roots.** *Skill source root* — where this file and the rest of Wayfinder's instructions live; always the Coclerk checkout, and requested for that purpose only. *Target repo* — the repo whose issue tracker this map and its tickets belong to: this session's already-connected project workspace, named before this skill was ever invoked — never the skill source root, even on the occasion the two are the same repo. If no project workspace is connected yet, or the connected one has no tracker of its own, ask the user which repo to work in and request that folder explicitly — never default to the skill source root. State the resolved target repo once, plainly, before the first tracker read or write.

The map is a single issue on the target repo's issue tracker, labelled `wayfinder:map`, the canonical artifact. Its tickets are child issues of the map.

The map is an **index**, not a store. It lists the decisions made and points at the tickets that hold their detail; a decision lives in exactly one place, its ticket, so the map never restates it, only gists it and links.

**Where the map, its child tickets, blocking, frontier queries, and tracker authentication physically live is tracker-specific.** The issue tracker should have been provided to you. If not, tell the user to run `/setup-matt-pocock-skills`. Consult the tracker doc's "Wayfinding operations" section for how _this_ repo (the target repo defined under **Two roots**, never the skill source root) expresses them, and follow its authentication bootstrap before any tracker reads or writes. If the tracker doc is missing, infer it: GitHub/GitLab remote (`git remote -v`) → that tracker; else local-markdown. Suggest `/setup-matt-pocock-skills` to record it.

**When the tracker doc's auth path fails, the tracker is not blocked — your transport is.** The map is a live artifact on a hosted tracker; never abandon the run, chart a parallel map elsewhere, or fall back to planning in chat merely because one access route didn't work. The tracker doc records _a_ route, and it may be stale or written for a different environment than the one you're in (a doc describing a CLI or `curl` recipe usually assumes a developer shell, not a sandboxed agent shell). Before declaring tracker access unavailable, exhaust the routes in order:

1. **The documented route** — the tracker doc's CLI or API recipe, with its authentication bootstrap.
2. **A connector** — an installed MCP server for the tracker, whose credentials are managed for you. Check what's connected before assuming there isn't one.
3. **The user's logged-in browser** — drive the tracker's web UI directly (in Cowork, the Claude in Chrome tools). Auth rides on the existing browser session, so this works precisely when token-based routes don't. Reading a map, its children, and their labels is a page read; claiming, commenting, and closing are UI actions. This route is slower per operation, so favour reading many tickets in one page load over one round trip each.
4. **The human** — only now. Say which routes you tried and what each did, and ask them to perform the tracker step.

Diagnose before escalating: a shell that reaches the tracker's host fine but fails only on authenticated calls is an environment restriction on carrying secrets, not an outage — jump straight to routes 2–3. Never work around such a restriction by disguising, re-encoding, or otherwise sneaking the credential past it. Once a route is known to work for a given repo, record it in the tracker doc so the next session starts there.

### The map body

The whole map at low resolution, loaded once per session. Open tickets are **not** listed: they are open child issues, found by query.

```markdown
## Destination

<what reaching the end of this map looks like: the spec, decision, or change this effort is finding its way to. One or two lines; every session orients to it before choosing a ticket.>

## Notes

<domain; skills every session should consult; standing preferences for this effort>

## Decisions so far

<!-- the index: one line per closed ticket, enough to judge relevance, then zoom the link for the detail the ticket holds -->

- [<closed ticket title>](link): <one-line gist of the answer>

## Not yet specified

<!-- see "Fog of war": in-scope fog you can't ticket yet; graduates as the frontier advances -->

## Out of scope

<!-- see "Out of scope": work ruled beyond the destination; closed, never graduates -->
```

### Tickets

Each ticket is a **child issue** of the map; the tracker's issue id is its identity. Its body is the question, sized to one 100K token agent session:

```markdown
## Question

<the decision or investigation this ticket resolves>
```

Each ticket carries a `wayfinder:<type>` label, one of `research`, `prototype`, `grilling`, `task` (see [Ticket Types](#ticket-types)). A `task` ticket additionally carries a HITL/AFK marker — exactly one of `wayfinder:hitl` or `wayfinder:afk`, set when the ticket is created — because `task` is the only type whose type label alone doesn't say whether a human is needed. The other three carry no marker: grilling and prototype are always HITL, research always AFK. A task found without a marker is treated as HITL — the safe default; add the missing marker when you encounter it. `wayfinder:` labels are the only labels a map and its tickets carry, never a triage label like `ready-for-agent`, which reads as "ready to implement": a map ticket's type and its HITL or AFK status come from its `wayfinder:` labels alone.

A session **claims** a ticket **first**, before any work, so concurrent sessions skip it: assign it to the map's dev. But parallel sessions may run as that same tracker account, so assignment alone can't tell two of them apart — assigning an already-assigned ticket to the same dev succeeds silently for both. So make the claim **verifiable**: after assigning, post a claim comment and re-read the ticket's claim comments. The tracker gives each comment its own id, and that id is the unique session id the claim needs — the **earliest** live one owns the ticket. Assignment events cannot serve, because a second assignment of an already-assigned ticket writes no event, so two near-simultaneous claims can share one. A session whose claim is not the earliest stands down and says so, leaving the assignment in place: it never unassigns, because the account is shared and unassigning would strip the winning session's claim along with its own. An open ticket carrying no live claim comment is unclaimed, whatever its assignee field shows. A claim idle 5 hours is stale: a HITL session asks before taking it over, as its own question; `afk` skips it. Claim ends in a blocker gate: a session that wins the claim but finds the ticket blocked reports it held with reason `blocked` and releases it at once, unlike `claimed`, which stops without unassigning. (The three steps — read, stamp, confirm — the gate, and how the claims are read are tracker-specific: see **Claim** under the tracker doc's "Wayfinding operations".)

Blocking uses the tracker's **native** dependency relationship: essential because it renders the frontier _visually_ in the tracker's own UI, so the human sees what's takeable without opening the map. Only a tracker that lacks native blocking falls back to a body convention. A ticket is **unblocked** when every ticket blocking it is closed; the **frontier** is the open, unblocked, unclaimed children, the edge of the known. Take from it in **issue-id order, lowest first** (oldest-charted first), so "the first frontier ticket in order" means the same thing to every session — unless the map's **Notes** declare a **ticket-selection priority** for the effort. A declared priority governs which frontier ticket is taken first, with id order breaking ties within it; it affects selection order only, never dependency edges. Blocking stated in prose (Notes, comments) must also be wired natively.

The answer isn't part of the body; it's recorded on resolution (see [Work through the map](#work-through-the-map)). Assets created while resolving a ticket are linked from the issue, not pasted in.

## Ticket Types

Every ticket is either **HITL** (human in the loop, worked _with_ a human who speaks for themselves) or **AFK**, driven by the agent alone. A HITL ticket only resolves through that live exchange; the agent never stands in for the human's side of it (a grilling agent that answers a question belonging to the human has broken this).

(When and in what shape to ask is `/grilling`'s Asking the user. The mechanics of presenting the evidence behind a decision — the source presented rather than quoted or described, full context rather than a curated slice, missing-data claims verified rather than inherited — are `/probanda:show-me`.)

- **Research** (AFK): Reading documentation, third-party APIs, or local resources like knowledge bases to surface a fact a decision waits on. Creates a markdown summary as a linked asset. Use when knowledge outside the current working directory is required.
- **Prototype** (HITL): Raise the fidelity of the discussion by making a cheap, rough, concrete artifact to react to (an outline, a rough take, a stub, or UI/logic code) by calling the Skill tool with "prototype". Links the prototype as an asset. Use when "how should it look" or "how should it behave" is the key question.
- **Grilling** (HITL): Conversation, resolved in two decision passes before the human is asked anything (see [Resolving a grilling ticket](#resolving-a-grilling-ticket)). The default case. Always call the Skill tool twice, for "grilling" and "domain-modeling".
- **Task** (HITL or AFK): Manual work that must happen before a _decision_ can be made: nothing to decide, prototype, or research, but the discussion is blocked until it's done. Signing up for a service so its API can be judged, provisioning access, moving data so its shape can be seen. This is the one type that _does_ rather than decides, and it earns its place by unblocking a decision, not by delivering the destination. The agent drives it alone where it can (AFK); otherwise it hands the human a precise checklist (HITL). Record that choice as a marker at creation — `wayfinder:afk` if the agent can complete it alone, `wayfinder:hitl` if it needs the human — so a background session can tell without reading the body. Resolved when the work is done; the answer records what was done and any resulting facts (credentials location, new URLs, row counts) later tickets depend on.

## Fog of war

The map is _deliberately_ incomplete: don't chart what you can't yet see. Beyond the live tickets lies the **fog of war**: the dim view of decisions and investigations you can tell are coming but can't yet pin down, because they hang on questions still open. Resolving a ticket clears the fog ahead of it, graduating whatever's now specifiable into fresh tickets, one at a time, until the way to the destination is clear and no tickets remain.

The map's **Not yet specified** section is where that dim view is written down: the suspected question, the area to revisit later. It's the undiscovered frontier _toward_ the destination: everything here is in scope, just not sharp enough to ticket. Write as loosely or as fully as the view allows; it doubles as a signpost for collaborators reading where the effort is headed.

**Fog or ticket?** The test is whether you can state the question precisely now, _not_ whether you can answer it now.

- **Ticket when** the question is already sharp, even if it's blocked and you can't act on it yet.
- **Not yet specified when** you can't yet phrase it that sharply. Don't pre-slice the fog into ticket-sized pieces: it's coarser than a ticket, and one patch may graduate into several tickets, or none, once the frontier reaches it.

**Not yet specified** excludes what's already decided (Decisions so far), what's already a live ticket, and what's out of scope (the next section).

**A stranded map.** Fog normally graduates when a ticket resolves. A map with **no open children** and text still under **Not yet specified** has no ticket left to resolve, so nothing will ever graduate that fog. Such a map is **stranded**. It is worked by [Re-chart the map](#re-chart-the-map), not by the normal loop. A map with open children is not stranded, even when its frontier is empty: the sessions holding those claimed or blocked tickets will clear the fog when they finish.

## Out of scope

Fog only ever gathers _toward_ the destination. The destination fixes the scope, so work beyond it is **out of scope**: it isn't fog, and it doesn't belong in **Not yet specified**. It gets its own **Out of scope** section on the map: work you've consciously ruled out of _this_ effort. Scope, not sharpness, lands it here.

Out-of-scope work never graduates (the frontier stops at the destination), so it returns only if the destination is redrawn, and then as a fresh effort, not a resumption.

Ruling something out of scope is a scoping act, not a step on the route. When a ticket that already exists turns out to sit past the destination (mis-scoped in while charting, or exposed by a resolution), **close it** (a closed ticket is unambiguously off the frontier) and leave one line in the **Out of scope** section: the gist plus why it's out of scope, linking the closed ticket. It stays out of **Decisions so far**, which records the route actually walked; a scope boundary isn't a step on it.

## Invocation

Three modes. Invoked bare, query the tracker for open maps: none → chart; one → work through it; several → ask the human, and route on what comes back (below). A named map or ticket bypasses this entirely — if it doesn't resolve there, say so and stop, never guess or switch away from the target repo (**Two roots**). Either way, **never work more than one ticket per session** — a fan-out dispatch (below) works zero. This body is already loaded by the slash command — never call the Skill tool for wayfinder.

**Several open maps.** In a chip-capable session (`spawn_task` available — the same test [Hand the map forward](#hand-the-map-forward)'s "No chip capability" uses), one `AskUserQuestion` (`multiSelect: true`): one option per open map, plus the tool's automatic free-text "Something else" for a new-map write-in. `AskUserQuestion` caps `options` at 4 (the write-in doesn't count against it); past 4 open maps, list the 4 lowest by issue id — matching the file's own lowest-first convention (see [Tickets](#tickets)) — name the rest by title as an aside, and point the human at `/wayfinder <map URL or number>` for any not listed. A session without chip capability asks the same way single-select — the write-in still works, since charting needs no chips — and never fans out.

Route on the answer:

- **Exactly one existing map, no write-in** → work through it, identical to today.
- **Two or more maps chosen** → **fan-out dispatch** — [Hand the map forward](#hand-the-map-forward)'s fan-out trigger; this session claims and resolves nothing itself.
- **A write-in alone** → this session is the destination session: route into [Chart the map](#chart-the-map), using that text as the loose idea.
- **A write-in with any picks** → fan-out dispatch for the picks, plus one destination chip for the write-in (see [Chart the map](#chart-the-map)). This session defines no destination, because it has done other work.
- **Nothing chosen** → stop.

`/wayfinder afk` is untouched by this: [Session mode](#session-mode-hitl-vs-afk) already stops an afk session facing several maps with none named — picking maps is a human question, and a chip still needs a human click — and that stands.

### Numeric-map continuation

An explicit decimal map number opts this session into a one-ticket chain. In a session that can create chips, a map number behaves as a bare invocation that resolved to that one map — work one ticket, then [Hand the map forward](#hand-the-map-forward), no successor; a map number always names a single map and never fans out. The successor chain exists only for sessions that cannot create chips. At invocation, capture the digits from `/wayfinder <map-number>` and derive the immutable successor prompt `invoke /wayfinder <map-number>`. Bare, URL, and `afk` invocations do not opt in and keep their existing stop behavior.

Launch a successor only after this session's claimed ticket is fully complete: its resolution is recorded, the map and newly surfaced tickets are updated, and required landing either succeeded or was correctly skipped. A HITL pause resumes in this session and launches nothing until the ticket later completes. Charting, re-charting, closing a map, finding no eligible ticket, losing the claim, any unresolved failure, or a landing conflict also launches nothing.

When an opted-in ticket completes, create exactly one fresh session as the final action:

- **Cowork:** create a one-off scheduled session for about one minute later with the exact successor prompt.
- **Codex:** create a separate task in the same local project with the exact successor prompt.
- **No fresh-session capability:** print the exact successor prompt for the human to invoke manually, then stop.

The fresh session re-reads the tracker; do not transfer conversation history, preselect its ticket, launch it as a subagent, or wait for it. After a successful launch, make no further tool calls. Never retry an ambiguous launch result, because that could create duplicate successors. A successor that finds no eligible ticket stops without launching another because it completed no ticket.

### Session mode: HITL vs AFK

Parallel sessions share one map, and a live human can only attend one at a time — so two sessions both reaching for a **HITL** ticket (grilling above all) collide on the human. An invocation argument keeps them out of each other's way when working through a map:

- **Bare `/wayfinder` — primary HITL session.** A human is at the keyboard. When picking a ticket, **prefer HITL tickets, grilling first**, and fall back to AFK tickets only when no HITL frontier ticket is takeable.
- **`/wayfinder afk` — AFK-only session.** No human is available to this session (a second parallel session, or a CI / scheduled / non-interactive run). It takes **only AFK tickets** — research, or a task marked `wayfinder:afk` — and **skips every HITL ticket**: grilling, prototype, and any task marked `wayfinder:hitl`. If the only takeable work is HITL, it takes nothing and stops.

The argument governs **ticket selection only**. It does not apply to charting: naming a destination is inherently HITL, so an `afk` invocation that would need a human — one that finds no map (charting is HITL), a map with no open children and fog left (re-charting is HITL), or several maps with none named to choose among — stops and says so rather than proceeding alone.

**A subagent cannot ask the human anything** (see [Subagent routing](#subagent-routing)), so the orchestrator is the only party that speaks to the human. A subagent that hits a material question stops and reports it — `planner` records it under its plan's assumptions, an implementer stops and reports the question, `routine-runner` stops at the failed step. In a session with a human, the orchestrator first decides any question the ticket, the map, or `/grilling`'s false-question classes settle, and records the decision in the next brief. It asks the rest by `/grilling`'s Asking the user, then dispatches a fresh subagent with the answers in its brief. In an afk session, the orchestrator answers only what the ticket or the map already decides. Otherwise it posts the question as a comment on the ticket, with the plan's path where a plan exists and the name of any uncommitted file a subagent left in the checkout, which it leaves in place. On a task it changes the marker from `wayfinder:afk` to `wayfinder:hitl`. A research ticket carries no marker, so the comment stands alone, and an afk session skips a ticket whose latest comment is such an unanswered question. It then releases the claim and stops — the same ending a failed escalation gets in an afk session. There is no blanket plan approval: a plan reaches the human only through the assumptions and open decisions the `planner` listed.

### Subagent routing

**The session is the orchestrator.** It runs on **Opus 5.5 at medium effort** by default, on every kind of ticket. **Fable 5.1 at low effort** is recommended instead for exactly two tickets: charting or re-charting a map, and a ticket whose whole deliverable is a written implementation plan, where the session briefs `planner`, reads the plan and records it, and builds, runs and checks nothing. A ticket that mixes either with anything else keeps the default. The recommendation is advisory: see the model check below. It talks to the human, reads and writes the tracker, claims, charts, re-charts, grills, briefs the subagents, runs every deterministic check itself, opens every file a subagent says it changed, commits, lands, and hands the map forward. It still works one ticket per session. A subagent never commits and never speaks to the human.

**Route per step, not per ticket.** A ticket's steps — planning, code, a scripted procedure, reading, review — are routed one at a time, each to the agent below. Model and effort are pinned in each agent's own definition, so this table names the agent and nothing else. The model in parentheses is what that definition says today, for the human reading the narration.

| Step | Agent |
|---|---|
| Write an implementation plan | `planner` (Fable 5.1) |
| Write or change code, run expected under about 30 minutes | `implementer` (Opus 5) |
| Write or change code, run expected to pass about 30 minutes | `implementer-long` (Opus 5) |
| Run a scripted, fully specified procedure | `routine-runner` (Sonnet 5) |
| Read and summarize sources into a findings file | `researcher` (Sonnet 5) |
| Review a code change, or review decisions | `reviewer` (Fable 5.1) |
| Answer a question the grilling profile delegates | `grilling-answerer` (Fable 5.1) — dispatched by `/grilling`, never from this table |
| Drive a web browser | `browser-driver` (Opus 5) |
| Sweep the tracker for the closing wave | `frontier-scout` (Sonnet 5) |
| Everything that needs the human or the tracker | the orchestrator |

**A small single-chain ticket stays in the orchestrator.** When the ticket is one dependent chain, fits this session's context, is fully specified, and leaves no design decision open, the orchestrator does it itself and dispatches nobody. No count of files, hours or ticket lines draws that line — the shape of the work does.

**Planning goes to `planner`.** The orchestrator reads the plan file itself. A plan gets no review pass.

**The orchestrator does not read implementation source before briefing.** It names the files, the specification and the check that proves the step in the brief, and lets the subagent read them. Pre-reading the code duplicates in the session's own context exactly what the subagent is about to read, and the orchestrator's context is the scarce one — it must still hold the plan, every diff and every check result afterwards. The tracker, the map, the plan and any specification the ticket names are not implementation source, so read those freely; and the verification pass after a subagent reports is not pre-reading either — that one opens the changed files by design.

**Code goes to `implementer`, or to `implementer-long` when the run is expected to pass about 30 minutes.** The orchestrator makes that call after reading the plan and names it with a one-line reason. Thirty minutes is the only line there is, and nothing in the ticket calibrates it, so say plainly that the length is an estimate.

**A scripted, fully specified procedure goes to `routine-runner`.** Never a long tool-driven run, and never a step with an open design decision left in it.

**Reading and summarizing goes to `researcher`.** The brief lists the sources to cover, and the orchestrator checks the findings file against that list. The `researcher` gathers and grades. The orchestrator reconciles the sources and writes the ticket's answer.

**Code from `implementer` or `implementer-long` gets one `reviewer` pass, after the deterministic checks pass.** A review never replaces a check. The orchestrator filters the reviewer's findings in its own pass and sends the accepted fixes to a fresh `implementer`, then re-runs the checks. The fixed code gets no second review, and a check that fails there goes to the human. `routine-runner` output and small single-chain work get no review pass.

**Browser driving goes to `browser-driver`, never the orchestrator.** The global CLAUDE.md rule on browser tasks governs: the brief carries the whole script with each step's expected result, one instance runs at a time, and the orchestrator makes a browser call itself only with the human's express permission for that one call. The human still does every sign-in. Browser output gets no review pass; the orchestrator checks the end state.

**Every subagent brief carries the full specification and the authorized scope up front** — the whole task, what is out of bounds, the check that proves the step, and "do not commit". Never disclose a brief over several turns.

**Escalate only on a failed deterministic check, once, in a fresh subagent.** When a step's named check fails, the orchestrator may re-dispatch that step one rung up: `routine-runner` to `implementer`, `implementer` to `implementer-long`. The retry is a new subagent, and its brief is the original specification plus the name of the check that must pass — never the failed attempt's report or its reasoning. A failure on `implementer-long` or on any agent with no rung above it, a second failure on any step, or a failure of a step that has no deterministic check goes to the human (in an afk session, see [Session mode](#session-mode-hitl-vs-afk)).

**When a named agent type does not exist in this environment**, say so once. A session with a human asks whether to do that step in the orchestrator instead, and the human's answer decides. An afk session ends as [Session mode](#session-mode-hitl-vs-afk) describes for an unanswerable question. A skill that dispatches its own agents, as `/grilling` does, follows its own fallback instead.

**A human can knowingly override any rule here.** The human's express instruction in this conversation, or their direct answer to the orchestrator's own question, overrides any routing rule for this conversation, and the orchestrator says once what was overridden. An override travels into no chip and no successor session. An afk session has no human, so it has no override.

**A map may set its own models.** An effort's **Notes** may declare the models for its own work — one model, or a choice among named models, per phase or per kind of ticket. A declared rule replaces the routing table **for that map only**. Notes rules govern subagent steps only: they never change the orchestrator's model, and the orchestrator's model check is unaffected. Read a Notes rule written per kind of ticket as applying to each matching step of such a ticket. A Notes rule may name which agent a kind of step goes to, or name a model for a step, which the orchestrator passes as that dispatch's own model value. Effort is not settable this way and stays as pinned, so a Notes rule may not name a model for an agent whose pinned effort is above that model's cap (Sonnet medium, Opus xhigh, Fable high). It names a different agent instead. If a dispatch does not accept a model value, Notes can only choose among the defined agents. A Notes rule may never make a reviewer weaker than the author it reviews. Distinct from a Notes-declared ticket-selection priority ([Tickets](#tickets)), which orders selection and never sets a model.

**The orchestrator's model check.** Before charting, re-charting or claiming, the session reads its running model from its own system context and compares it with what the ticket calls for. On that model it proceeds silently. On anything else, a session that cannot tell which model it is on included, it says once which model the ticket calls for, then proceeds. It never asks and never stops, and an `afk` session is no different. Effort cannot be read from the session, so the effort named above is for the human who opens the session and is not checked. A session whose model changes after it has claimed says so once and finishes its ticket rather than abandoning it mid-flight.

Why the routing falls this way. `planner` runs on Fable 5.1 on the strength of an internal planning A/B, not a public measurement, so give no other reason for it. The orchestrator runs on Opus 5.5 by default because a measured execution ticket put the session's own tokens overwhelmingly into verification the skill refuses to delegate. It runs at medium effort because that is the maintainer's stated default for Opus 5.5 (2026-09-24). Fable 5.1 is recommended only where the session's whole output is one written artefact and it verifies nothing, a map or a plan, on the strength of the same internal A/B that pins `planner`, so give no other reason for it. A grilling ticket stays on the default, on a mechanism rather than a measurement: its hard questions are the technical ones the human's grilling profile delegates, so Fable already answers them in session as `grilling-answerer` and reconsiders every decision afterwards as the `reviewer` at the mandatory end review. Charting has no such route. The destination question is a scope decision matching neither profile scope, so by `/grilling`'s own rule it goes to the human and Fable sees it only at the end review; fog mapping is autonomous, with the same end-review-only pass; and ticket slicing happens after grilling ends, with no Fable pass at all. That is why charting wants Fable holding the session and a grilling ticket does not. One equal-tier cost is already paid on purpose: `grilling-answerer` answers at the same tier as the `reviewer` that reconsiders it, buying a better first answer at the price of a less useful review of it. What a step actually needs is visible only once the plan exists — the same ticket can hide a one-line typo or a multi-module refactor — so the model is chosen per step, not per ticket. Code is where the cheap tier measures far behind the top two, so it never runs cheap. Reading and summarizing is where the cheap tier measures level with them, and more thinking there buys more invented detail rather than more coverage, so research runs cheap and the orchestrator does the reconciling. The closing sweep runs cheap on the same shape: its queries are recipes the tracker doc already spells out, and its judgment is one tier call and one overlap call per ticket, while the orchestrator's context at the close is the scarce one. A review pays only when the reviewer is at least as strong as the author, so review sits at the top tier and no weaker reviewer is substituted for it. That holds by tier only, not once effort is counted, and the reviewer cannot run tests. A failed attempt carried into a retry's context lowers the retry's accuracy, so an escalation starts clean. And no report from a subagent is a completion check: agents misreport incomplete work often, and no model judges task success reliably, so the orchestrator runs the checks and opens the changed files itself.

### Chart the map

User invokes with a loose idea. Charting takes two sessions: the first only defines the destination, and the second maps the frontier as a re-chart.

**A Destination is written only in a session devoted to defining it.** That session does nothing else, before or after. A session that has done any other work, or holds a ticket, never creates a map and never writes or changes a Destination, even when the human states an intent mid-session. It spawns one destination chip instead: `title` `CHART: Define the destination for <idea>`, `cwd` the target checkout, `prompt` beginning `/wayfinder`, then the loose idea and any words of the human's quoted verbatim, labelled as input to the grilling. The prompt carries no drafted destination. Changing an existing map's Destination follows the same rule.

1. **Name the destination.** Call the Skill tool twice, for "grilling" and "domain-modeling", with the human. A statement of intent, such as "the plugin is meant for Cowork", is input to that grilling and never becomes a destination by itself. Before writing, show the human the destination's concrete scope: what it includes, roughly how many skills, steps or items it covers, what existing behaviour it changes or blocks, and what it excludes. Write it only after the human's explicit yes to that wording.
2. **Create the map** (label `wayfinder:map`): Destination, Out of scope and Notes filled in, Decisions so far empty, and one line under **Not yet specified**: "The frontier: not yet mapped." With no children and fog left, the map is stranded, so the next session re-charts it.
3. **Spawn one re-chart chip** (the one fan-out spawns for a stranded map, under [Hand the map forward](#hand-the-map-forward)) in place of the closing ceremony, and stop. Map no frontier and create no tickets.

The frontier session is [Re-chart the map](#re-chart-the-map). If it shows the whole journey fits one session, no map is needed: close the map with a comment saying so, and ask the human how to proceed.

### Re-chart the map

A map is **stranded** (see [Fog of war](#fog-of-war)) when it has no open children but still has text under **Not yet specified**. Re-charting is the grilling half of charting, with the destination already fixed. Like charting, it is one session's whole work: it claims no ticket and resolves none.

1. **Load the map and confirm it is stranded.** An `afk` session does not re-chart: it stops here and says the map needs a re-chart. Query its open children. If any child is open, this is not a re-chart: go back to [Work through the map](#work-through-the-map) and pick from the frontier as usual.
2. **Grill over the fog, breadth-first.** Take every item under **Not yet specified**, item by item, calling the Skill tool twice, for "grilling" and "domain-modeling", and test each against the fixed Destination. Bring each item to one of three ends: it becomes one or more tickets; it is ruled **out of scope**; or the task or research that would make it sharp enough to ticket becomes a ticket itself. An item stays in the fog only when the human says to park it.
3. **Update the map.** Create the new tickets as child issues of the map — each with its `wayfinder:<type>` label, and every `task` with its `wayfinder:hitl` or `wayfinder:afk` marker — then wire blocking edges in a second pass (issues need ids before they can reference each other). Write cross-references in that pass too, with real ids: a placeholder `#<n>` auto-links to an unrelated issue. Remove from **Not yet specified** every item that became a ticket or was ruled out. A ruled-out item gets one line under **Out of scope**. Re-fetch the map body and confirm the changes landed.
4. **Run [Hand the map forward](#hand-the-map-forward)** over the tickets just created that were born unblocked — re-charting counts as charting there — then stop. Do not also resolve tickets. If the human parked every item, say plainly that the map stays stranded by choice and nothing was dispatched.

### Close the map

A map with no open children and nothing left under **Not yet specified** is **complete**: the destination has been reached, or every step still short of it was ruled out of scope. Post one comment on the map issue saying that every child is closed and no fog remains, then close the map. Nothing is decided here, so any session mode may do it — an `afk` session and a fan-out session included. Closing a map completes no ticket: it launches no successor and dispatches nothing. A closed map no longer appears when a bare `/wayfinder` queries for open maps.

### Work through the map

User invokes with a map (URL or number), or one was discovered. A ticket is **optional**: without one, you pick the next decision, not the user.

1. Load the **map**: the low-res view, not every ticket body.
2. Choose the ticket. If the user named one, use it. Otherwise pick from the frontier by **session mode** (see [Session mode](#session-mode-hitl-vs-afk)): a **HITL** session takes the first frontier ticket in order, preferring HITL tickets and grilling ahead of the rest; an **AFK** session takes the first AFK frontier ticket in order — research, or a task marked `wayfinder:afk` — skipping every HITL ticket (grilling, prototype, and any task marked `wayfinder:hitl`). Where the map's Notes declare a ticket-selection priority, it governs the choice among mode-eligible tickets, ahead of the default type preference and id order. **If the map has no open children at all**, there is no ticket to choose. If text is left under **Not yet specified**, a HITL session goes to [Re-chart the map](#re-chart-the-map), and an `afk` session stops and says the map needs a re-chart. If nothing is left there, go to [Close the map](#close-the-map). Either one replaces the rest of these steps. If no ticket qualifies for this mode, stop and say so rather than taking a HITL ticket the session can't work. Never ask the user which ticket — these rules decide. Then run the [orchestrator's model check](#subagent-routing) and **claim it** before any work — the verifiable claim under [Tickets](#tickets), run by the tracker doc's **Claim**; if it refuses (`claimed`), stop without unassigning; if it reports `blocked`, Claim has already released the ticket, so stop and say so.
3. Resolve it as the type its `wayfinder:<type>` label names (see [Ticket Types](#ticket-types)). Read the label, not just the body: the body never states the type. **Zoom as needed**: fetch the full body of any related or closed ticket on demand; call the Skill tool for whichever skills the `## Notes` block names. Route each step of the work by [Subagent routing](#subagent-routing). If in doubt, call the Skill tool twice, for "grilling" and "domain-modeling". A grilling ticket resolves through [Resolving a grilling ticket](#resolving-a-grilling-ticket). Never ask permission to do work the ticket already instructs unconditionally. If unticketed work blocks it, ticket that work, wire the edge, then follow [Hand the map forward](#hand-the-map-forward)'s blocker-ticketed trigger — it releases this ticket, then dispatches a chip for the new ticket, and ends the session. Releasing first means the ticket carries no live claim by the time the blocker can close. Unticketed work that does *not* block it is filed as an issue rather than narrated, by the tangent rule in `shared/closing-ceremony.md`. **Before writing the resolution, read this ticket's own comment thread**, not just its body — a concurrent session may have already posted a decision on it; if one has, reconcile explicitly by superseding it in a single governing comment rather than silently overwriting (a decision lives in exactly one place).
4. Record the resolution: post the answer as a **resolution comment**, **close** the issue, and **append a context pointer** to the map's Decisions-so-far, then re-fetch the map body and confirm it landed. If the human reports the work was done outside the map, say so and name who confirmed it.
5. Add newly-surfaced tickets (create-then-wire, labelling each with its type and every task with its `wayfinder:hitl`/`wayfinder:afk` marker); graduate any fog the answer has made specifiable, clearing each graduated patch from **Not yet specified** so it lives only as its new ticket. If the answer reveals that a ticket (this one or another) sits beyond the destination, **rule it out of scope** rather than resolving it on the route. If the decision invalidates other parts of the map, update or delete those tickets.
6. **Land the work (cloud sessions only).** If this session is a remote/cloud execution environment and the ticket left committed repo changes, finish by landing the effort branch on `main` — see [Landing work in cloud sessions](#landing-work-in-cloud-sessions).
7. **Hand the map forward.** Only after steps 1–6 fully succeed, follow [Hand the map forward](#hand-the-map-forward) — computed only now, so the chips start from a tracker and `main` that already reflect this ticket. In a session that cannot create chips, it falls back to [Numeric-map continuation](#numeric-map-continuation) for an opted-in map.

The user may run unblocked tickets in parallel, so expect other sessions to be editing the tracker concurrently.

### Resolving a grilling ticket

A grilling ticket runs `/grilling` as written, which already runs two decision passes before the human is asked anything: pass 1 walks the whole design tree and defers every question that belongs to the human, pass 2 is the End review, and the remaining questions go to the human in one list. A wayfinder grilling ticket adds what follows, and overrides Finish's "Do not ask the user to confirm the summary" in the two cases steps 3 and 4 name — no question left but the end review revised a decision, or something changed after the answers — so one confirmation comes before anything is posted to the tracker, asked in `/grilling`'s Asking the user format. When the end review ran and revised nothing, no confirmation is asked, and the two-line notice step 3 names is the one exception to Asking the user's "nothing else is red".

1. **Pass 1 — the orchestrator.** Walk the design tree per `/grilling`, with no wayfinder override.
2. **Pass 2 — the end review.** Run `/grilling`'s End review as written. A wayfinder grilling ticket adds four things to it:
   - **Brief content.** The brief also carries the ticket body and its comments, and the map's Decisions-so-far.
   - **Premises.** The reviewer also reconsiders the **premises** the session relied on: the ticket body's stated intent and the closed-ticket resolutions it built on. An intent recorded before a later resolution changed its premise is never a governing decision on its own.
   - **Reclassification.** The reviewer re-checks each decision's classification, asking whether an autonomous or delegated one belonged to the human.
   - **Batching.** A recommended change to a closed ticket's resolution, and any decision reclassified as the human's, is **not** applied. It joins `/grilling`'s end list with the reviewer's recommendation.
3. **End the turn with the summary, then the list**, per `/grilling`'s Finish. When both passes leave no question:
   - **and the end review ran and revised no decision** (its "Decisions the reviewer revised" list in the Finish summary is "none", or holds only revisions that leave each outcome unchanged), do not ask. After the summary and the line saying the grilling session is done, show these two lines, then continue as if the summary had been confirmed:
     `$\color{red}{\text{Reviewer approved all decisions.}}$`
     `$\color{red}{\text{Confirmation ceremony skipped.}}$`
     Any revision of a decision's outcome counts, a wording fix included. A revision that changes only a decision's classification or reasoning, leaving its outcome unchanged, does not count, though the summary still lists it. These two lines are the one red text that is not a question.
   - **and the end review revised a decision**, ask one closing question instead: confirm the summary before anything is posted. (When no reviewer was available, `/grilling`'s End review step 5 already asks this, so nothing here changes.)
4. **After the answers**, per `/grilling`'s Finish. If anything changed, show the changed items and ask one confirmation before posting; otherwise post. An approved change to a closed ticket's resolution is recorded on that ticket as a superseding comment.

Then record the resolution per step 4 of [Work through the map](#work-through-the-map).

### Hand the map forward

Dispatch is decentralized: no session orchestrates the whole map. Each session hands the map forward — spawning a chip session (Claude Code's `spawn_task` — `mcp__ccd_session__spawn_task`, however narrowly its own description reads — a session the user starts with one click) for whatever's now takeable — then ends. A chip is how the next orchestrator session starts: a subagent can't talk to the human, and a map is by definition more than one context holds, so each ticket gets its own fresh session. Three of its four triggers are a closing act, spawning chips for whatever the session's own work just made takeable — a fully completed ticket, a charted map, or a blocker ticketed for the session's own in-progress ticket; the fourth (fan-out, below) is a dispatch-only session's entire act, spawning chips for a frontier the session didn't create. No session monitors the chips it creates, in any trigger.

**When it runs.** As the closing act of a session that fully completed its claimed ticket, finished charting a map's frontier, which is always a re-chart (a destination session spawns only its one re-chart chip, under [Chart the map](#chart-the-map)), or ticketed a genuine blocker for its own in-progress ticket and wired it — **blocker-ticketed dispatch** (the "If unticketed work blocks it..." step under [Work through the map](#work-through-the-map)). "Fully completed" is a self-audit, run before dispatching: resolution comment posted, naming the commit where work landed — or, when the work was done outside the map, naming who confirmed it; issue closed; the map's Decisions-so-far line re-fetched and confirmed — the common miss; newly surfaced tickets created and wired; landing done or correctly skipped. A re-chart session's self-audit is shorter: new tickets created and wired; **Not yet specified** cleared of every item that became a ticket or was ruled out; the map body re-fetched and confirmed. A session that stopped early for any *other* reason — paused for a HITL human, hit a landing conflict — dispatches nothing. A session that finished a planned slice of a multi-session ticket has not stopped early: it hands off by the closing ceremony's **Planned hand-off** (`.claude/skills/shared/closing-ceremony.md`). Releasing a claim specifically because a blocker was ticketed and wired is not one of these early-stop cases: it leaves the tracker in a clean, fully consistent state rather than a failed or incomplete one, which is exactly what blocker-ticketed dispatch fires on. Session mode doesn't change any of this: the `afk` flag governs ticket selection only.

One trigger fires outside a session's close, at its *start*: **fan-out dispatch**, when [Invocation](#invocation)'s bare-invocation picker got two or more maps chosen (or a write-in alongside any picks). The self-audit above, and the "stopped early → dispatches nothing" rule, apply only to the completed-ticket and charted-map triggers — a fan-out session completes no ticket and holds no claim, so there's nothing to audit or to have stopped early on, and blocker-ticketed dispatch carries its own check instead (the blocker ticket must be created, wired, and born unblocked — not a completion audit).

**The dispatch is the closing ceremony.** Once the self-audit passes, a completed-ticket or charted-map trigger computes no dispatch set of its own: it runs `shared/closing-ceremony.md` (read with `mcp__plugin_probanda_data__skill_source__read`), which owns the sweep, the ranking, the cap, the overlap hold, the dedupe, the chip prompts and the tangent rule. The closing narration adopts that doc's signal contract — the turn ends with the chips, then exactly one of its fixed last lines, and nothing after it. The two triggers that spawn chips without closing a ticket keep their chips and each end with their own distinct line from the same contract, so a chip alone never reads as a ticket close: blocker-ticketed dispatch with `Released #<n> <title>, blocked by #<k>.`, fan-out with `Fan-out: <k> chips.`

For **fan-out dispatch**, the set is scoped to the maps the human chose rather than swept from both queues: the combined current frontier — every open, unblocked, unclaimed child (see [Tickets](#tickets)) — of each map the human chose, uncapped. This is the one trigger whose entire job is dispatching tickets already sitting on the frontier. A chosen map with an empty frontier gets a closer look. **No open children, and fog left under Not yet specified** → the map is stranded: spawn one **re-chart chip** — `title` `CHART: Re-chart <map title>`, `cwd` the target checkout, `prompt` beginning `/wayfinder <map URL>`, then a context block saying that this is a re-chart of a stranded map, the checkout's absolute path, the tracker doc, the model line (Fable 5.1 at low effort, since re-charting is charting), and that the chip claims no ticket and ends by handing the map forward in turn. **No open children, and nothing under Not yet specified** → run [Close the map](#close-the-map) here and now, and say so in the closing narration. **Open children, all claimed or blocked** → say so in one sentence; the sessions holding them will move the map on. Name each re-chart chip in the closing narration beside the ticket chips, marked HITL.

**Query failure is not ticket failure.** The ticket is already closed; never reopen or stall it. If the dependents query fails after exhausting the transport routes under [The Map](#the-map), say plainly that dispatch was skipped and why, then stop — the map self-heals, since any later bare `/wayfinder` works the frontier.

**Then finish.** Spawning the chips is the session's final act: no monitoring, no reconciliation, no waiting on chips. Never ask whether to dispatch, or offer to keep working instead — not even a "keep going?" aside. An unclicked chip simply sits visible in the app until the human clicks it. State plainly which chips were created and why, then end.

A fan-out dispatch can leave several HITL chips sitting at once with no queue between them — the old one-HITL-at-a-time hold stays gone, by design. Say so plainly in the closing narration, so the human knows they're serial work for one person, but build no queue: an unclicked chip still just sits until clicked, same as any other.

**No chip capability** (`spawn_task` unavailable — CI, scheduled, or another non-chip harness), in order: a session opted into [Numeric-map continuation](#numeric-map-continuation) that completed a ticket follows that section instead — one successor as the final action, no printed prompts, since the successor re-reads the tracker itself (a re-chart completes no ticket, so it prints its prompts like any other session); otherwise print each prompt the ceremony returned, in rank order, in its own fenced block, then the signal contract's last line and stop. A chip-capable session never launches a successor — chips win outright.

### Landing work in cloud sessions

A cloud/remote wayfinder session works on a **designated effort branch**, not `main`, so a resolved ticket that touches the repo never reaches `main` on its own — the decided work piles up unmerged. To close that gap, a cloud session **lands each ticket on `main` itself**. The human is never asked to merge.

Try the direct route first: merge `origin/main` into the effort branch and push it to `main` (`git push origin HEAD:main`). If the harness refuses a push outside the designated branch, that guardrail stands and this step works within it: push the effort branch, open a PR into `main`, and merge it yourself once its checks pass. Never force-push.

**Scope — only cloud sessions.** This applies only when your session context identifies a **managed remote/cloud execution environment** (Claude Code on the web, or a scheduled / CI run) working on a designated branch off `main` — not the user's local machine. A local session works on `main` and opens no PR. A local session that finds an open effort-branch PR from a cloud session merges it once its checks pass, without asking. Chips land on `main` instead (see [Hand the map forward](#hand-the-map-forward)).

**When to skip.** Run this only when the ticket actually **committed repo changes**. Pure planning tickets — grilling, research, prototype — touch only the tracker and the map issue, so there is nothing to land; skip straight to finishing. Also skip when a PR isn't **appropriate**: the effort's **Notes** say to hold the branch for a single end-of-map merge, or the committed changes don't stand on their own and would leave `main` broken mid-sequence. When in doubt about appropriateness, leave the work on the branch and say so.

**Before you open the PR, refresh from `main` and reconcile.** A cloud effort branch can be many commits behind `main`, and `main` may have moved in ways that overlap this effort — the same file added, the same problem solved a different way, the change made moot. So fetch `origin/main` and compare **before** opening the PR: if `main` already carries an equivalent change, **drop the redundant local commit** rather than propose a duplicate; if `main` took a different approach that supersedes this work, **stop and surface it to the human** rather than fight it. Only propose what is still genuinely needed on top of current `main`.

**When it is appropriate:**
1. Ensure the ticket's changes are committed on the effort branch and the branch is pushed.
2. Merge `origin/main` into the effort branch and push it to `main` (`git push origin HEAD:main`). If the push is accepted, report the commit in the wrap-up and in the ticket's resolution comment, and skip steps 3 to 5.
3. If the push is refused: if a PR from the effort branch into `main` is already open, it now carries this ticket's commits too; do not open another. Otherwise open one (`gh pr create --base main --head <effort-branch>`), titled and described by the ticket.
4. Merge the PR yourself (`gh pr merge --merge`) once its checks pass. Report the PR link in the wrap-up and in the ticket's resolution comment, then switch nothing: later work continues on the effort branch.
5. **PR cannot be opened or merged** (no `gh`, no permission) → leave the work on the pushed branch, report the branch name or PR link in one line, and finish. The next local session lands it.

The one-ticket-per-session rule is unchanged — resolve one ticket, finish its landing, hand the map forward, then finish.
