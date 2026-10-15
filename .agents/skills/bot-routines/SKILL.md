---
name: bot-routines
description: Create, list, change, pause, and delete the bot's routines (T3 scheduled tasks on a time, interval, or webhook), and save a repeatable process as a skill. Use when the user wants something to happen regularly or at a time ("every weekday at 9, check X", "remind me Friday"), asks what is scheduled, or says "save this as a skill".
---

# bot-routines

A routine is a standing responsibility: a T3 scheduled task that wakes you in the home thread with saved instructions. A skill is how to do a job; a routine is when to do it. Get a job right once, save it as a skill, then hang it on a routine.

## Create a routine

Pin these down, in one exchange at most:
- What to do, and the report bar: what is worth telling the user. A routine without a bar reports noise.
- When: `{"type":"fixed_time","timeOfDay":"09:00","weekdays":[1,2,3,4,5]}` (local time; 0 is Sunday), `{"type":"interval","everyMs":...}` (at least 15 minutes), or `{"type":"webhook"}` for an event (report the returned `webhookUrl`; the URL alone is the only key, so for anything beyond reporting require a signature, with the secret from `request_secret`).
- Approval boundary: what the routine may do alone, what it must bring back.

Then:
1. Routines bind to the thread that creates them, and they belong in the home thread (see `me/HEARTBEAT.md`; your own thread id is `parentThreadId` from `orchestrator_capabilities`). If you are not in the home thread, `t3_thread_send` the home thread one message that starts with exactly `Create routine:` (that is how the home thread recognizes it), `Create routine: <name> | <schedule> | <instructions> | bar: <bar> | never: <limits>`, tell the user it is being set up there, and stop. Otherwise `list_scheduled_tasks` to avoid duplicates, then continue.
2. `schedule_task` with `title` `<bot name>: <routine name>`, `bindToCurrentThread: true`, and a prompt in this shape:
   ```
   [routine] <name>
   Read .agents/skills/bot-heartbeat/SKILL.md (the [routine] part) first.
   <instructions, or "Read .agents/skills/my-<skill>/SKILL.md and follow it.">
   Report bar: <what is worth reporting>
   Never: <what needs approval>
   ```
3. Record it under Routines in `me/HEARTBEAT.md` (name, schedule in words, scheduledTaskId, bar).
4. Confirm in one line with the next run time. Offer `run_scheduled_task_now` to test it; a test run does real work.

Runs use the home thread's model. If a routine needs a different model, its instructions should hand the work to a worker (`bot-delegate`) and only summarize.

One-off reminders ("remind me Friday at 3"): a `fixed_time` routine named `reminder-<what>`, prompt starting `[routine] reminder-<what>`, whose instructions end with "then disable this routine with update_scheduled_task enabled=false". Convert the user's timezone to the machine's local time (`date`) and say both.

## Standard routines

Offered at onboarding; create only what the user approved.
- morning brief (daily, user's morning): `[routine] morning-brief` - greet in your voice; what is due today; in-flight work status; the "Ideas" from NOW.md (refreshed nightly by reflect); at most one open question from USER.md. Always report.
- nightly reflect (daily, before the user's sleep or late night): `[routine] reflect` - follow the bot-reflect skill. Report only changes the user should know about.
- evening wrap (optional): what got done, what is waiting on them tomorrow.

## Manage

- List: `list_scheduled_tasks`, cross-check with `me/HEARTBEAT.md`, fix drift in the file.
- Change: `update_scheduled_task` (prompt, schedule, title, enabled). Pause rather than delete unless asked.
- Delete: `delete_scheduled_task` only when the user asks; remove the line from `me/HEARTBEAT.md`.

## Save a process as a skill

When a job went well and will recur, or the user says "save this as a skill":
1. Name it `my-<kebab-name>` and write `.agents/skills/my-<name>/SKILL.md`:
   ```
   ---
   name: my-<name>
   description: <what it does>. Use when <trigger>.
   ---
   # my-<name>
   ## When to use
   ## Inputs and access
   ## Steps
   ## Validate
   ## Return
   ## Needs approval
   ```
2. Fill it from what you actually did, including the failure handling you discovered. Keep it under 100 lines.
3. Tell the user, and offer a routine if it recurs on a schedule.
