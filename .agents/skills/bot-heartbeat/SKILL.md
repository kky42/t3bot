---
name: bot-heartbeat
description: What the bot does on each scheduled [heartbeat] or [routine] run - check in-flight work, due follow-ups, and the checklist, take one small initiative, and stay silent unless something needs the user. Also how to create or change the heartbeat schedule. Use when a message starts with [heartbeat] or [routine], or when setting up or tuning the heartbeat.
---

# bot-heartbeat

The heartbeat is what makes you a bot instead of a chat: a T3 scheduled task that wakes you in your home thread on a cadence, even when the user is away. Most heartbeats should end with nothing to say.

## On a `[heartbeat]` run

1. Read `me/HEARTBEAT.md`, `me/NOW.md`, and the Rhythm section of `me/USER.md`. Do not reread everything else unless a step needs it.
2. Quiet hours: if now is inside them, do only steps 3 and 4 and stay silent unless something is urgent by the user's own definition.
3. In flight: for each item, check its worker thread with `t3_thread_read` (small `limit`). Finished: verify the result against its "check" before believing it, then move it to "Waiting on the user" with a one-line summary and evidence. Stuck, failed, or waiting on a pending request: say so precisely.
4. Follow-ups: do the ones that are due, or say why not.
5. Run the checklist in `me/HEARTBEAT.md`.
6. Initiative, at most one new job per day and only outside quiet hours: if an open loop, or an idea the user has already seen in a brief, can be advanced without the user (research, a draft, a check), hand it to a cheap worker (`bot-delegate`) and record it under "In flight". Never start anything external or irreversible.
7. Update `NOW.md`. Append one line to today's log only if something happened.

Reply:
- Nothing needs the user: `HEARTBEAT_OK` and nothing else, or `HEARTBEAT_OK - started <thing>` if you started something. No status recap; the user reads these at a glance.
- Something needs the user: no `HEARTBEAT_OK`. Lead with what they must decide or know, one line each, in your own voice, then stop.

Never repeat a message you already sent about the same item unless its state changed.

## On a `[routine] <name>` run

The prompt carries the routine's instructions and report bar. Follow them. If a source is unavailable, report the failure instead of using stale data. Reply `ROUTINE_OK <name>` and nothing else when nothing meets the bar; otherwise report in your voice. Log one line.

## Set up or change the heartbeat

Runs inherit the provider and model of the thread they post into, so cost depends on the home thread's model. Check it with `t3_thread_configuration` and tell the user if it is an expensive one.

1. Read the home thread from `me/HEARTBEAT.md`. Outside it (`orchestrator_capabilities` → `parentThreadId` differs), hand it over like any routine (`bot-routines` step 1) and stop.
2. `list_scheduled_tasks` first; never create a second heartbeat. Change an existing one with `update_scheduled_task`.
3. Create it with `schedule_task`:
   - `title`: `<bot name> heartbeat`
   - `schedule`: `{"type":"interval","everyMs":3600000}` (default; the user's choice wins; never under 15 minutes)
   - `prompt`: exactly `[heartbeat] Follow the bot-heartbeat skill.` Put user-specific checks in the checklist in `me/HEARTBEAT.md`, never in the prompt, so they can change without touching the schedule.
   - `bindToCurrentThread`: true
4. Record the `scheduledTaskId` and cadence under Routines in `me/HEARTBEAT.md`. Tell the user the next run time.

To pause: `update_scheduled_task` with `enabled: false`, note it in `me/HEARTBEAT.md`.
