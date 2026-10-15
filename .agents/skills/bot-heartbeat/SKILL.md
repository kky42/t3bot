---
name: bot-heartbeat
description: What the bot does on each scheduled [heartbeat] or [routine] run - check in-flight work, due follow-ups, and the checklist, take one small initiative, and stay silent unless something needs the user. Also how to create or change the heartbeat schedule. Use when a message starts with [heartbeat] or [routine], or when setting up or tuning the heartbeat.
---

# bot-heartbeat

The heartbeat is what makes you a bot instead of a chat: a T3 scheduled task that wakes you in your home thread on a cadence, even when the user is away. Most heartbeats should end with nothing to say.

## On a `[heartbeat]` run

1. Run `bin/beat --heartbeat` and do what its `SKILL CHANGED`, `GOALS`, and `CADENCE` lines say, silently. It prints the time, quiet hours, when the user was last here, and work in flight; never guess those. Read `me/HEARTBEAT.md` and `me/NOW.md`; nothing else unless a step needs it.
2. Quiet hours: if now is inside them, still do steps 3 and 4 (checks never pause), skip 6, and stay silent unless something is urgent by the user's own definition.
3. In flight: for each item, check its worker thread with `t3_thread_read` (small `limit`); for a peer job, read the peer's home thread too, in case it answered there instead of sending. Items marked `for: <peer>` that are done but unsent: send the result now (`bot-team`). Finished: verify the result against its "check" before believing it, then move it to "Waiting on the user" with a one-line summary and evidence. Stuck, failed, or waiting on a pending request: say so precisely.
4. Follow-ups: do the ones that are due, or say why not.
5. Run the checklist in `me/HEARTBEAT.md`.
6. Goals and initiative, at most one new job per day and only outside quiet hours: if the script printed `GOALS: missing`, interview the user about this bot's long-term goals - propose 1-3 derived from its role and the user's goals in `me/USER.md`, get their confirmation or edits, and write them under `## Goals` in `me/HEARTBEAT.md`. Otherwise, if a goal can be advanced by a small job a worker could do (research, a draft, a check), hand it to a worker (`bot-orchestrate`) and record it under "In flight". Never start anything external or irreversible.
7. Update `NOW.md`, write the `- Next:` line in `me/HEARTBEAT.md` (see Cadence), and append one line to today's log only if something happened.

Reply:
- Nothing needs the user: `HEARTBEAT_OK` and nothing else, or `HEARTBEAT_OK - started <thing>` if you started something. No status recap; the user reads these at a glance.
- Something needs the user: no `HEARTBEAT_OK`. Lead with what they must decide or know, one line each, in your own voice, then stop.

Never repeat a message you already sent about the same item unless its state changed.

## Cadence

The heartbeat is the bot's outer loop: each run reconciles state (in flight, follow-ups, checklist, goal progress), acts on at most one thing, then sets its own next wake. Three mechanisms:

- **Goals** (`## Goals` in `me/HEARTBEAT.md`, 1-3 lines): the long-term directions that make proactivity purposeful. Every run considers whether one small, non-blocking job can advance one (the initiative rules above apply). No goals means no direction; the script flags a bot that has none.
- **Next wake** (`- Next: YYYY-MM-DDTHH:MM reason ...` in `me/HEARTBEAT.md`): at the end of every run, estimate when the next reason to act exists - a follow-up coming due, in-flight work likely finished, a goal step worth attempting, a watch likely to change - and write that line. `bin/beat` sleeps until then (clamped 15m-720m); rewrite it whenever the estimate changes materially. Dynamic factors (kky, 2026-10-15): how far along the goal is, how much effort the next step needs, whether the bot is waiting on user feedback (slow down, batch), and how urgently the user wants the goal pursued (speed up). Waiting on the user is the most common reason to sleep long.
- **Backstop** (the Base line): when Next is missing, stale, or unparseable, `bin/beat` falls back to the fixed ladder - base while the user was active in the last 2 days, 4h floors after 2 days away, 12h after a week. The fixed interval is a safety net, not the rhythm; never under 15 minutes.

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
4. Record it under Routines in `me/HEARTBEAT.md` as `- heartbeat | every <n>m | scheduledTaskId: <id>`, and the interval as Base and Current under Cadence. Tell the user the next run time.

To pause: `update_scheduled_task` with `enabled: false`, note it in `me/HEARTBEAT.md`.
