---
name: bot-heartbeat
description: What the bot does on each scheduled [heartbeat] or [routine] run - check in-flight work, due follow-ups, and the checklist, take one small initiative, and stay silent unless something needs the user. Also how to create or change the heartbeat schedule. Use when a message starts with [heartbeat] or [routine], or when setting up or tuning the heartbeat.
---

# bot-heartbeat

The heartbeat is what makes you a bot instead of a chat: a T3 scheduled task that wakes you in your home thread on a cadence, even when the user is away. Most heartbeats should end with nothing to say.

## On a `[heartbeat]` run

1. Run `bin/beat --heartbeat` and do what its `SKILL CHANGED` line says, silently; a `GOALS` line is for step 6. It prints the time, quiet hours, when the user was last here, and work in flight; never guess those. Read `me/HEARTBEAT.md` and `me/NOW.md`; nothing else unless a step needs it.
2. Quiet hours: if now is inside them, still do steps 3 and 4 (checks never pause), skip 6, and stay silent unless something is urgent by the user's own definition.
3. In flight: for each item, check its worker thread with `t3_thread_read` (small `limit`); for a peer job, read the peer's home thread too, in case it answered there instead of sending. Items marked `for: <peer>` that are done but unsent: send the result now (`bot-team`). Finished: verify the result against its "check" before believing it, then move it to "Waiting on the user" with a one-line summary and evidence. Stuck, failed, or waiting on a pending request: say so precisely.
4. Follow-ups: do the ones that are due, or say why not.
5. Run the checklist in `me/HEARTBEAT.md`. When a heartbeat run prints `DOCTOR: due`, run `bin/doctor` and handle every line: fix FAILs that are file edits you are allowed to make (`me/` files only), put anything else under "Waiting on the user" in `NOW.md`, and never skip the two `DOCTOR: T3-CHECK` comparisons (they are exact compares via `list_scheduled_tasks`, zero judgment).
6. Goals and initiative, only outside quiet hours:
   - `GOALS: missing - ... propose`: propose 1-3 long-term goals for this bot, each with a measure (see Cadence), derived from your role in `SOUL.md` and the user's goals in `USER.md`, as one message to the user; add `- goals asked <date>` under "Waiting on the user" in `NOW.md`. Write them under `## Goals` in `me/HEARTBEAT.md` when the user answers. Ask once: `GOALS: missing - goals asked ...` means wait.
   - Otherwise, at most one new job per day: if a step toward a goal, an open loop, or an idea the user has already seen in a brief can be advanced without the user (research, a draft, a check), hand it to a worker (`bot-orchestrate`) and record it under "In flight". Never start anything external or irreversible.
7. Update `NOW.md`. Append one line to today's log only if something happened.
8. Set your next wake (see Cadence): rewrite the `- Next:` line in `me/HEARTBEAT.md`, then run `bin/beat --apply-next` and do what its `NEXT` and `CADENCE` lines say, silently, in this run.

Reply:
- Nothing needs the user: `HEARTBEAT_OK` and nothing else, or `HEARTBEAT_OK - started <thing>` if you started something. No status recap; the user reads these at a glance.
- Something needs the user: no `HEARTBEAT_OK`. Lead with what they must decide or know, one line each, in your own voice, then stop.

Never repeat a message you already sent about the same item unless its state changed.

## Cadence

The heartbeat is your outer loop: each run reconciles state, acts on at most one thing, and sets its own next wake. Never mention cadence changes unless asked.

- **Goals** (`## Goals` in `me/HEARTBEAT.md`, 1-3 lines): the long-term directions that make initiative purposeful, each written `- <goal> — measure: <how progress is judged>`. A goal without a measure cannot be optimized: use a number or an observable state where possible, an explicit qualitative test otherwise. Set at onboarding; `bot-reflect` marks finished ones `(done YYYY-MM-DD)` and proposes replacements.
- **Next** (`- Next: YYYY-MM-DDTHH:MM <reason>` under Cadence, local time): your estimate of when the next reason to act exists - a follow-up coming due, a worker likely finished, a goal step worth attempting. Weigh how far along the goal is (judged by its measure, and cite it in the reason), how much effort the next step needs, whether you are waiting on the user (sleep longer and batch), and how urgently the user wants it (wake sooner). Never a routine's run time (the routine wakes you itself; `bin/beat` ignores such a Next), never inside quiet hours unless urgent (landing at their end is fine).
- **How `bin/beat --apply-next` turns that into the interval**: a future Next sets it, clamped to 15m-720m and, while the checklist has items, to at most Base, so checks never go dark. Without a usable Next (missing, past, unreadable) it uses the backstop: Base while the user was here in the last 2 days, at least 4h after 2 days away, at least 12h after a week. While "In flight" has a line naming a `worker:`, `task:`, `peer:`, `thread:` or `for:`, it wakes at least every 60m (workers notify the thread on completion; this is a backstop). It asks for a change only past max(15m, 20% of Current), so small drifts do not rewrite the schedule. `bin/wake` never reads Next; it only tightens the interval when the user is back or work is in flight.
- After applying, check the task's next run in the `update_scheduled_task` result: T3 may count the new interval from the last run rather than now, so it can land up to one old interval off Next. If it lands well after a Next that matters, say so in `NOW.md` and check again next run.

## On a `[routine] <name>` run

The prompt carries the routine's instructions and report bar. Follow them. If a source is unavailable, report the failure instead of using stale data. Reply `ROUTINE_OK <name>` and nothing else when nothing meets the bar; otherwise report in your voice. Log one line.

## Set up or change the heartbeat

Runs inherit the provider and model of the thread they post into, so cost depends on the home thread's model. Check it with `t3_thread_configuration` and tell the user if it is an expensive one.

1. Read the home thread from `me/HEARTBEAT.md`. Outside it (`orchestrator_capabilities` → `parentThreadId` differs), hand it over like any routine (`bot-routines` step 1) and stop.
2. `list_scheduled_tasks` first; never create a second heartbeat. Change an existing one with `update_scheduled_task`.
3. Create it with `schedule_task`:
   - `title`: `<bot name> heartbeat`
   - `schedule`: `{"type":"interval","everyMs":3600000}` (default; the user's choice wins; never under 15 minutes)
   - `prompt`: exactly `[heartbeat] Run bin/beat --heartbeat, then do every step of .agents/skills/bot-heartbeat/SKILL.md; quiet hours mute messages, not checks.` Long-lived threads keep old instructions in context; `bin/beat` says when the skill changed. Put user-specific checks in the checklist in `me/HEARTBEAT.md`, never in the prompt, so they can change without touching the schedule.
   - `bindToCurrentThread`: true
4. Record it under Routines in `me/HEARTBEAT.md` as `- heartbeat | every <n>m | scheduledTaskId: <id>` (keep the label: `bin/beat` reads it), and the interval as Base and Current under Cadence. Tell the user the next run time.

To pause: `update_scheduled_task` with `enabled: false`, note it in `me/HEARTBEAT.md`.
