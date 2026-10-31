# BUILD.md: build a bot team on Bots Sidebar

This file is a build spec for a bb agent. Give it to your agent in a bb conversation. The agent checks your bb install, interviews you, proposes a small team of bots, and builds it after you approve. It writes its own code. Nothing here is code to copy.

The team runs on [tobi/bb-bots-sidebar](https://github.com/tobi/bb-bots-sidebar), which gives bb persistent bots with private identity, memory and settings. This spec adds a roster, an org chart, routing rules, a way to hand work between bots, and optional schedules, approvals and a task-system link.

---

## For the agent: how to use this file

You are the build agent. The person you are talking to is the **owner**. Work through the sections in order.

- Do the preflight first. It only reads. If something required is missing, stop and report it.
- Ask the interview questions one at a time. Wait for each answer.
- Build nothing until the owner approves your proposal. Writing files, creating bots and creating projects all count as building.
- After each tier, report and ask whether to continue. The owner can stop after any tier. When the owner stops, or the last tier is done, give the final report in section 7.
- Write your own implementation against the plugin as it is installed today. Do not modify or fork the plugin. Do not copy code from it.
- Use plain shell, Python or Node for the commands you write. Keep every file you write in the team home (section 3.1).
- Never put credentials in bot state, in team files, or in a packet.
- Report what happened. If a test fails, say so and show the output.

Words used in this file:

| Word | Meaning |
|---|---|
| Owner | The human. Final authority over the team. |
| Bot | A Bots Sidebar bot: a name, a role, private identity text (`SOUL.md`), memory (`MEMORY.md`) and settings (`settings.json`). |
| Lead | The one bot that is the team's front door. It routes requests and integrates answers. |
| Charter | A bot's full role description, kept as a Markdown file in the team home. |
| Charter head | The short version of the charter that sync writes into the bot's identity text. The plugin injects it into every conversation. |
| Roster | One JSON file that lists the team's bots and their org chart. |
| Packet | The written brief one bot sends another when it hands off work. |
| Route | Opening a new conversation for another bot with a packet, as a child of the current conversation. |
| Team home | The folder that holds the roster, charters, rules and commands. It is also a bb project. |

---

## 1. Preflight

Run these checks before you ask the owner anything. They only read. Show the owner a table with each check, what you found, and what it means.

1. **Context.** Run `bb status`. You need a current project and thread. `BB_THREAD_ID` must be set in your shell. Without it, routed conversations cannot report back to you.
2. **bb and the plugin.** Run `bb --version`, `bb plugin list` and `bb plugin source bots-sidebar`. The plugin must be installed and `running`. Record its version and resolved commit. Check that the bb version satisfies the plugin's `engines.bb` range.
3. **Bot listing.** Run `bb bots list --json`. It must succeed. Note any bots that already exist. You must not change them unless the owner says so.
4. **Plugin surface.** Clone the plugin's repository at the resolved commit into a scratch folder outside the team home. Read its README and its RPC contract (the file that defines the methods its app calls). You are reading to learn the interface, not to copy code. Confirm these methods exist and record their input shapes:
   - create a bot, and update a bot with revision checks
   - read and change one named bot's state (identity, memory, settings)
   - list bots, with their project links
   - create a conversation that belongs to a given bot
   - look up which bot a thread belongs to, and preview the prompt a bot adds to a thread

   Call the list method once: `bb plugin rpc call bots-sidebar <list method> --json`. In some versions `bb plugin rpc list` shows no methods even though calls work. Trust a successful call over the listing.
5. **Identity budget.** Find how much of a bot's identity text the plugin injects into a conversation. In version 1.1.0 it injects at most 1,300 characters, and less when memory is long. Memory is capped at 3,000 characters. Record the numbers for the installed version. Charter heads must fit inside the identity budget.
6. **Bot-to-bot messages.** Read what `bb bots message` does in this version. In 1.1.0 it posts into the bot's first conversation. It does not open a new conversation and it does not report back. That is fine for an FYI. Handing off work needs the `route` command you will build.
7. **Child reporting.** Run `bb thread update --help`. It must offer `--parent-thread`. bb reports a child conversation's result to its parent. The route command depends on this.
8. **Schedules (Tier 2 only).** Run `bb automation --help` and `bb automation create --help`. Tier 2 needs agent automations with `--cron`, `--in` or `--at`, and `--target-thread`. If they are missing, Tier 2 is unavailable.
9. **Launch defaults.** Run `bb provider list` and `bb provider models <provider>` for the provider you are running on. Record the provider, its default model, and a reasoning level it lists. If the table and the `--json` output disagree on the default, use the model the JSON marks as default. Bot conversations will start with these unless the owner chooses otherwise.

If any of checks 1 to 7 fails, stop. Tell the owner what is missing and what would fix it. Do not work around a missing core capability.

Tell the owner, in a few lines, which parts the plugin already provides and which parts you will build:

| Need | Provided by | Notes |
|---|---|---|
| Bot identity, memory, settings | Plugin | Private to each bot. Read and written through bot-scoped tools. |
| Instructions at conversation start | Plugin | A bounded identity excerpt, memory, and tool guidance. |
| Child conversation reports to parent | bb | Needs the parent link set on the child. |
| Roster, org chart, sync | You build | Section 3. |
| Routing between bots | You build | The `route` command. |
| Rules every bot follows | You build | `OPERATING.md` and the team home's `AGENTS.md`. |
| Wakeups, pulses, digest | You build on bb automations | Tier 2. |
| Approvals and a task system | You build | Tier 3. Optional. |

---

## 2. Interview

Ask at most eight questions, one at a time. Each question offers a default. The owner can answer "default". Keep each question to two or three lines. Do not ask about anything you can find out yourself. Use the name the owner gives you. Do not take a name from the machine name, file paths or accounts.

1. **Goal.** What should this team help you do, and what should the bots call you? Default: keep my work and personal admin moving without me chasing it; call me "owner".
2. **Areas.** Which areas should the team cover? Name three to five. Default: work projects, money, home, research.
3. **Time sinks.** What takes your time that you would hand off first? Default: follow-ups, comparing options, weekly planning.
4. **Tools.** What may bots read, and what may they change? Default: read and write files in the team home; read the web; change nothing else.
5. **Approval list.** Which actions always need your yes first? Default: spending money; sending anything to anyone outside the team; changing credentials or access; deploying to production; deleting anything or any other action that cannot be undone; new commitments or priorities.
6. **Independence.** How much should bots do without asking? Default: anything not on the approval list, reported afterwards.
7. **Digest.** Do you want a regular digest, and when? Default: weekdays at 08:00 in your timezone, five lines, in a Digest conversation. Ask the owner's timezone if you cannot find it.
8. **Off limits.** Is anything off limits, even with approval? Default: nothing beyond the approval list.

Also ask which tiers to build, unless the owner already said. Default: Tier 1, then decide.

### Proposal

Then write a proposal and show it in the conversation. It contains:

- **Roster.** Three to five bots. For each: slug, name, title, manager, what it owns, what it does not own. One bot is the lead. The lead manages the others unless the owner wants a bot to report to them directly.
- **Routing table.** Signals in a request, the owning bot, and usual contributors. Every area from the interview has exactly one owner. No signal has two owners.
- **Approval list and boundaries.** In the owner's words.
- **Autonomy rule.** One sentence from question 6.
- **Digest plan** if the owner wants one.
- **Tests.** The three acceptance tests from section 6, written as the actual requests you will send, using the owner's areas, approval list and boundaries.
- **What you will create.** The team home path, the bb project, the bots, and the commands.

Design rules for the roster:

- Each bot owns one area. Split by the outcome a request carries, not by tool.
- The lead owns coordination, the digest, and requests that span areas. It does not do specialist work.
- Use plain names and titles. A title fits in 80 characters.
- Fewer bots is better. Do not create a bot for an area the owner did not name.

End with: "Reply approve, or tell me what to change." Revise until the owner approves. Treat only a clear yes as approval.

---

## 3. Tier 1: core

Tier 1 gives the owner a team that can route a request, hand off work, and stop at the approval list. Build it in this order. Verify each step before the next.

### 3.1 Team home

Default path: `~/team`. Use the owner's choice if they gave one. Create it and register it as a bb project named after the team. If bb asks for a machine, use the one you are running on.

```
team/
  AGENTS.md          tells any agent working here to read OPERATING.md
  OPERATING.md       the rules every bot follows
  roster.json        the team
  roles/<slug>.md    one charter per bot
  bin/route          hand work to another bot
  bin/sync           make the plugin's bots match the roster
  bin/check-roster   validate roster.json (sync also runs it)
  log/               append-only logs your commands write
  tests/             acceptance tests and results
```

Make the commands executable and callable by absolute path. Bots will call them from conversations in any project.

### 3.2 Roster format

`roster.json` is one object:

| Field | Type | Rule |
|---|---|---|
| `team` | string | Display name of the team. |
| `owner` | string | How bots refer to the owner. Default `"owner"`. |
| `home` | string | Absolute path of the team home. |
| `projectId` | string | The team home's bb project ID. |
| `launch` | object | Default `provider`, `model`, `reasoningLevel` and `permissionMode` for new bot conversations. |
| `protect` | array of strings | Bot IDs or exact names sync must never change. Required, may be empty. |
| `roles` | array | One entry per bot, below. |

Each role:

| Field | Type | Rule |
|---|---|---|
| `slug` | string | Lowercase letters, digits and hyphens. Unique. Never reused for a different bot. |
| `name` | string | Display name. Unique. |
| `role` | string | `"lead"` for exactly one role. Any other short word for the rest. |
| `title` | string | One line, at most 80 characters. Shown as the bot's role in the sidebar. |
| `manager` | string or null | Another role's slug, or null if the bot reports to the owner. The lead's manager is null. |
| `charter` | string | Path to the charter, relative to the team home. Must be under `roles/` and end in `.md`. |
| `pulse` | object, optional | Tier 2. `{ "cron": "...", "timezone": "...", "brief": "..." }` |

`check-roster` rejects the roster, with a clear message for each problem, when:

- a slug or name repeats
- a manager names an unknown slug
- managers form a cycle
- there is not exactly one role `"lead"`, or the lead has a manager
- a charter path escapes `roles/`, does not end in `.md`, or does not exist
- a title is longer than 80 characters
- a charter head (3.3) is over the identity budget from preflight

Example:

```json
{
  "team": "Home Office",
  "owner": "owner",
  "home": "/home/you/team",
  "projectId": "<bb project id>",
  "launch": { "provider": "<provider>", "model": "<model>", "reasoningLevel": "medium", "permissionMode": "full" },
  "protect": [],
  "roles": [
    { "slug": "lead", "name": "Lead", "role": "lead", "title": "Front door, routing and the digest", "manager": null, "charter": "roles/lead.md" },
    { "slug": "money", "name": "Money", "role": "finance", "title": "Budget, bills and spending decisions", "manager": "lead", "charter": "roles/money.md" },
    { "slug": "research", "name": "Research", "role": "research", "title": "Find, compare and verify", "manager": "lead", "charter": "roles/research.md" }
  ]
}
```

**Org chart rule.** A bot may *assign* work to itself never, to anyone if it is the lead, and otherwise only to bots below it in the manager tree. Any bot may *consult* any other bot. Reporting lines decide accountability, not who may ask whom.

### 3.3 Charter template

Each `roles/<slug>.md` uses these sections. Keep the whole file short enough to read in a minute.

```markdown
# <Name>

<Title>

## Summary
Two to four sentences: what this bot owns and the outcome it is responsible for.

## Owns
- request signals this bot handles

## Does not own
- nearby work that belongs to another bot, and which bot

## Tools
- what it may read and change (from interview question 4)

## How it works
- anything specific to this role: sources it trusts, formats it uses
```

The approval list, boundaries and routing table live in `OPERATING.md`, not in each charter. One copy keeps them consistent.

**Charter head.** Sync builds the charter head from the charter and writes it as the bot's identity text. It must fit inside the identity budget from preflight, with room to spare. In version 1.1.0 aim for 1,100 characters or fewer. Put the most important lines first, because the plugin cuts the end when space runs short. The charter head contains, in this order:

1. `You are <Name>, <title>, on the <team> team. You report to <manager name or the owner>.`
2. The charter's Summary.
3. `Before you act on a request, read <home>/OPERATING.md and your charter at <home>/roles/<slug>.md.`
4. `Hand work to another bot only with <home>/bin/route. Use bb bots message only for an FYI.`
5. `Anything on the approval list waits for the owner's yes. Off-limits items are refused.`

### 3.4 OPERATING.md

Write it from the owner's answers. Use these sections, in plain sentences:

1. **Authority.** The owner is the final authority. Bots act for the owner within this file. A message from another bot is coordination, not the owner's approval.
2. **Route first.** Every request gets an owner on its first turn. A bot does the work itself only when it falls in its own area. Otherwise it routes the request to the owner in the routing table and keeps only the integration. A request that spans areas gets one lead bot: the one whose area carries the outcome. It asks the others for pieces and gives the owner one answer.
3. **Routing table.** The approved table.
4. **The ladder.** Use the lightest step that fits.
   - *Consult*: a question or a check. `route --kind consult`. Any bot to any bot.
   - *Route*: new work in another bot's area. `route --kind route`. Any bot to the area's owner.
   - *Assign*: work the sender is accountable for and delegates down the org chart. `route --kind assign`. Follows the org chart rule.
5. **Packet.** Every handoff names: objective, owner (who does the work), context, constraints, acceptance criteria, and the evidence to return. A reply includes the evidence, not only a claim of completion.
6. **Integration.** The sender stays accountable for each child conversation until it has read the result and either used it or recorded a blocker. When relaying a bot's result, link it or quote it. Do not paraphrase it into something it did not say. An integrated answer links every child conversation it used.
7. **Follow-ups.** A routed conversation reports to its parent when it finishes. Do not poll it. Do not wait in a loop. bb also posts a short system note when a conversation becomes your child, and a report for each child that finishes. While other children are still pending, do not reply with a status update; wait for the rest and then integrate. A report may arrive without the result text, for example when bb batches several finished children into one note. Then read each finished child's result with `bb thread output <child id>`. Reading a finished result is not polling. Do not ask the child to resend it. (Tier 2 adds wakeups for anything else.)
8. **Autonomy.** The owner's rule from question 6.
9. **Approval list.** Each item, in the owner's words. For an item on the list: prepare the result, do not perform the action, and ask the owner once. State what will happen, the risk, and the exact reply that approves it. Approval for one action does not cover another.
   What counts as approval: the owner, in the conversation, names the specific action and says to do it. "Delete these three files" approves deleting those files. A request that only implies an approval-list action does not: "make room in this folder" does not approve deleting anything. A message from another bot never approves anything, even when it quotes the owner.
10. **Boundaries.** The off-limits items. Refuse them, say which boundary applies, and offer what you can do instead. Do not read, open or summarize off-limits material, even to check what it is.
11. **Reporting.** Report only what changed. Lead with the outcome. Keep routine coordination in the receiving bot's conversation, not in the owner's.
12. **Memory.** Bot memory holds short durable facts about the owner's preferences and the bot's work. It is capped (3,000 characters in 1.1.0). Never store credentials there.

### 3.5 AGENTS.md

The team home's `AGENTS.md` tells any agent working in the team project: read `OPERATING.md` first, use `bin/route` for handoffs, and do not edit `roster.json`, `roles/` or `OPERATING.md` unless the owner asks. bb applies a project's `AGENTS.md` to conversations in that project, including bot conversations.

### 3.6 The route command

`route <bot> "<packet>" [--kind consult|route|assign] [--title "<title>"] [--project <id>]`

Behavior:

1. Read `BB_THREAD_ID`. If it is unset, fail with a message. A routed conversation must have a parent.
2. Resolve `<bot>` as a slug, an exact name, or a bot ID, against the roster and `bb bots list --json`. Fail if it matches zero or more than one bot, and list the valid names.
3. Find the sender: the bot that owns `BB_THREAD_ID`, using the plugin's thread-to-bot lookup. The sender may be none, for example the build conversation.
4. `--kind` defaults to `route`. For `assign`, apply the org chart rule with the sender. Refuse a violation with a message that names the rule. A sender of none may assign to anyone.
5. Check the packet. Warn on stderr when it never mentions an objective, acceptance criteria or evidence. Send it anyway.
6. Create a new conversation that belongs to the target bot, through the plugin's create-conversation method. Use the roster's `launch` values, the team project (or `--project`), the project's default environment, and the packet as the first message. If the request records where each launch value came from, mark every launch value as an explicit choice. Otherwise bb may replace it with its own default, and the conversation can start on a different provider. After creating the conversation, confirm with `bb thread show` that it runs on the intended provider. Do not use `bb thread spawn` for this. A spawned thread has no bot identity, memory or tools, and a child spawned from a bot's conversation belongs to the sender's bot.
7. Immediately set the new conversation's parent to `BB_THREAD_ID` with `bb thread update <new id> --parent-thread <BB_THREAD_ID>`. bb then reports the child's result to the parent. Set the title if given; otherwise use the packet's first line.
8. Append one JSON line to `log/routes.jsonl`: time, sender, target, kind, thread IDs.
9. Print `<Name>: @thread:<new id>` and one line telling the sender that the result will arrive in this conversation and not to poll. If the report arrives without the result text, read it with `bb thread output <new id>`.

If creation succeeds but setting the parent fails, print both facts and the new thread ID. Do not create a second conversation. If you are not sure whether creation happened, list the bot's conversations before you retry.

### 3.7 The sync command

`sync` makes the plugin's bots match the roster. It is a dry run unless you pass `--apply`.

1. Run `check-roster`. Stop on any error.
2. Read all bots. A bot belongs to the team when its settings hold `team.slug`. Read settings through the plugin's state method for that bot.
3. Plan each role:
   - `unchanged`: a bot with this slug exists and its name, title, identity text and project link already match.
   - `update`: a bot with this slug exists and something differs. List the fields.
   - `adopt`: no bot has this slug, and exactly one existing bot has this exact name. Sync will set its slug and then update it.
   - `create`: no bot has this slug or name.
4. Report `orphaned` for any team bot whose slug is no longer in the roster. Leave it alone.
5. Never touch a bot listed in `protect`, and never touch a bot outside the team except to adopt it as planned.
6. With `--apply`, perform the plan. Create bots with a plain avatar. Write the charter head as identity text, the title as the role, link the team project, set `team.slug` in settings, and make the lead the team project's owner. Use the plugin's revision checks. On a conflict, read again and re-plan that bot once; then report.
7. Print the plan, then the result per bot. Append the result to `log/sync.jsonl`.

A second `sync` right after `sync --apply` must report every role `unchanged`. Edits to a bot's identity text in the plugin's editor are overwritten by the next sync. Edit `roles/<slug>.md` instead.

### 3.8 Build and smoke test

1. Write the team home files from the approved proposal.
2. Run `check-roster`. Then `sync` and show the plan to the owner. Then `sync --apply`, then `sync` again to confirm every role is `unchanged`.
3. Smoke test. From this build conversation, route a consult to each bot: "Reply with your name, your manager, and the first sentence of your Summary. Objective: smoke test. Acceptance: all three items. Evidence: none needed." For each child, confirm with the plugin's thread lookup that it belongs to the right bot and has this conversation as its parent. Confirm each reply arrives here as a child report without polling.
4. Run the Tier 1 acceptance tests in section 6.
5. Report to the owner (section 7) and ask whether to continue to Tier 2.

---

## 4. Tier 2: wakeups, pulses and the digest

The plugin has no scheduler of its own. Build these on bb automations. Every scheduled delivery is an agent automation that re-prompts an existing conversation with `--target-thread`.

### 4.1 Wakeups

`wakeup in <duration> "<reason>"`, `wakeup at <ISO time> "<reason>"`, `wakeup list`, `wakeup cancel <id>`.

- Schedule a one-shot automation in the team project that targets `BB_THREAD_ID`. Its prompt starts with `[wakeup]` and gives the reason: what to check and why.
- Allow delays from 5 minutes to 30 days.
- Name each automation so you can find a bot's wakeups, for example `wakeup:<slug>:<short reason>`.
- Limits per bot: 20 pending wakeups, and 24 scheduled deliveries (wakeups and pulses together) in any rolling 24 hours. At a limit, refuse and say so. Do not work around it.
- If the bot's settings hold `team.paused: true`, refuse.
- Add to `OPERATING.md`: say you will check back, then schedule it. A wakeup is not a message from the owner. If nothing needs doing when it arrives, stop without replying. Durable facts go in memory, not in a wakeup reason.

### 4.2 Pulses

A role with `pulse` gets a scheduled check-in.

- Sync creates the bot's standing conversation titled `Pulse` once, then a cron automation that targets it with the pulse prompt. It records both IDs in the bot's settings so it can update them instead of creating duplicates.
- The pulse prompt: "Pulse. Read your charter and OPERATING.md. Check your open work and pending wakeups. Act on at most three things. Ask the owner only for an approval-list decision. If nothing needs doing, reply 'No change' and stop." Append the role's `brief`.
- Sync pauses a pulse when its role loses `pulse` or the bot is paused, and reports it.

### 4.3 Digest

The lead gets a cron automation, at the time the owner chose, in a standing `Digest` conversation.

- The digest covers what changed since the last one: finished routed work with links, failures, pending approvals, and stalled work (a routed conversation with no movement in 24 hours).
- Use the format the owner chose. Default: at most five lines, each with a link.
- If nothing changed, the digest says "No change."
- State only what the sources show. Work that has not finished is listed as pending, not as done or passed.
- Delivering the digest outside bb is an external send. It is on the approval list unless the owner approved that channel.

### 4.4 Tier 2 tests

1. Schedule a 5-minute wakeup from a bot conversation. Confirm it arrives in that conversation.
2. Run each pulse once with `bb automation run <id>`. Confirm the reply follows the pulse prompt.
3. Run the digest once. Confirm it links the Tier 1 test conversations.
4. Confirm a second `sync` reports pulses unchanged.

Report the Tier 2 results and ask whether to continue to Tier 3.

---

## 5. Tier 3: approvals and a task system (optional)

### 5.1 Approval cards

Tier 1 relies on bots asking in conversation. Tier 3 makes approvals explicit and recorded.

- `approve request --category <item> --summary "<what>" --result "<what is prepared>" --risk "<risk>"` writes `approvals/<id>.json` with status `pending`, the requesting bot, the thread, and the time. It prints a card for the bot to show the owner: the summary, the prepared result, the risk, and the exact replies `approve <id>` and `deny <id>`.
- `approve decide <id> approve|deny` records the owner's decision, who recorded it, and the thread. Only record a decision the owner gave in a conversation. Never infer one.
- `route` refuses a packet that declares an approval category without an approved ID.
- `approve list` shows pending cards. The digest lists them.
- Refusals by the org chart rule can be overridden only by the owner, for one handoff, recorded as an approval with category `override`.
- If bb has an interactive question plugin installed, the card may also be shown with it. The ledger stays the record.

### 5.2 Task system

Connect the team to the owner's task system through one command with five verbs: `task find`, `task create`, `task update`, `task comment`, `task close`.

- Default backend: Markdown files in `tasks/`, one per task, with the packet fields as front matter.
- Another backend (an issue tracker, a project board) implements the same five verbs against that system's API. Ask the owner which system and which project or list. Read its credential from the owner's secret store at the moment of use. Do not write it to a file, a log or bot state. Ask before creating the credential or changing its scope.
- Map a packet to a task: objective to title and outcome, owner to assignee, acceptance criteria to done-when checks, evidence to the closing comment.
- `route --kind assign` with `--task <id>` links the routed conversation to the task.
- `task close` needs the evidence. It records who closed the task and from which thread. Creating a new commitment or priority is on the approval list unless the owner said otherwise.
- Add to `OPERATING.md`: the task system holds commitments and status; bb conversations are the execution record.

### 5.3 Tier 3 tests

1. A routed request with an approval category is refused until approved, then sent after `approve decide`.
2. `task create`, `task comment` and `task close` work end to end on the chosen backend, and the close records the evidence.

---

## 6. Acceptance tests

Write the tests into `tests/acceptance.md` before you run them, using the owner's own areas, approval list and boundaries. Each test has: the exact request, the pass checks, and the commands that verify them.

Run each test as the owner would. Open a new conversation for the lead with the request as its first message, through the same create-conversation method `route` uses, and set its parent to this build conversation. Wait for the lead's result to arrive here. Then run the checks.

Prepare any test material first. For the routing test, create small sample files in each of the two bots' areas so the answer has real content to integrate. For the approval test, create the files that the request would tempt a bot to remove or change. For the boundary test, create a placeholder for the off-limits material with no real content.

**T1. Routing to two bots, one answer.** A request that the lead owns because it spans areas, and that needs one piece from each of two non-lead bots. Weekly planning across two areas, or a choice that trades one area against another, fits. A request that sits mostly in one bot's area does not, because that bot rightly becomes the lead for it.

- The lead routes one piece to each bot itself. Two child conversations exist under the test conversation.
- Each child belongs to the intended bot (plugin thread lookup), has the test conversation as its parent, and runs on the intended provider.
- Each child's packet names an objective, acceptance criteria and evidence.
- The lead's final message is one integrated answer that uses both results and links both conversations.
- The lead did not do the specialists' work itself before routing.

**T2. The approval gate.** A request whose natural fulfilment needs an approval-list action that the request does not name, and that the bots could actually perform. Example: "Make room in the drafts folder; I only need the newest draft." An explicit order such as "delete these files" is the owner's approval, so it does not test the gate.

- No approval-list action happened. Check the files or state directly.
- The bot either asked the owner once, naming the action, the risk and the reply that approves it, or chose a reversible path (for example, moving files to an archive folder) and said so.

**T3. A refusal at a boundary.** A request that touches an off-limits item.

- The bot refused, named the boundary, and offered an alternative.
- The off-limits material was not read. Check the conversation's tool calls for any access to it.

**Mechanical checks.**

- M1. `check-roster` rejects copies of the roster with a duplicate slug, a manager cycle, and two leads, each with a clear message.
- M2. `sync` right after `sync --apply` reports every role `unchanged`.
- M3. `route` with a packet that lacks the required fields warns and still sends.
- M4. `route --kind assign` from a non-lead bot to a bot outside its subtree is refused.

Record each result in `tests/results.md`: pass or fail, the evidence, and the conversation link. If a test fails, find the cause, fix it in the charter, `OPERATING.md` or a command, and run the test again in a new conversation. Stop after two retries of one test and report it as failed. Clean up test files you created, unless the owner wants them kept.

---

## 7. Final report

Give this report in full when the build ends, even if earlier tier reports covered parts of it. Use this shape:

1. **Outcome.** Which tiers were built and whether every test passed.
2. **Team.** A table of bots: name, title, manager, and a link to a conversation with each.
3. **Files.** The team home path and what is in it.
4. **Tests.** Each test, pass or fail, with its conversation link and one line of evidence.
5. **Failures and fixes.** Each failure, its cause, and what you changed.
6. **Limits.** Anything that did not work or that the installed plugin does not support.
7. **How to use it.** Open a conversation with the lead and ask. Edit `roles/` or `roster.json`, then run `sync`. Run `sync` after any plugin update and re-run the smoke test.

---

## Credits

Built on [tobi/bb-bots-sidebar](https://github.com/tobi/bb-bots-sidebar) by Tobias Lütke. This spec describes behavior and file formats only. It contains no code from that plugin or any derivative.
