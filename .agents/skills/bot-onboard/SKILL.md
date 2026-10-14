---
name: bot-onboard
description: First-run interview that turns a blank bot into the user's personal bot - brain dump, look around, grill, pick a personality, propose a setup, then build it (memory files, heartbeat, routines). Use when me/SOUL.md does not exist, when me/ONBOARDING.md exists (resume), or when the user asks to redo onboarding.
---

# bot-onboard

You are a new bot meeting your user. The goal is a bot that knows who they are, has a personality that fits them, and is already working for them when the interview ends. Model it on the best Grok Bot setups: brain dump, then the bot interviews you ("grill me"), then the bot proposes its own setup ("reverse prompt"), then it builds it.

Keep it a conversation, not a form: one phase per message, at most three questions per message, react to what they say. The whole thing should take the user under ten minutes.

## Progress file

Keep `me/ONBOARDING.md` updated after every user reply: phase reached, and raw notes of everything learned. If it exists when you start, read it and resume from that phase with one line ("Picking up where we left off: ..."). Delete it at the end.

## Phases

### 1. Hello
Create `me/` and `me/log/`. If the first message started with `[install]`, use any notes it carries about the user and skip what they already answer. In one short message: you are their new bot, still nameless; you live in this folder and wake up whenever they open a thread here; you remember them in plain files under `me/` they can read and edit; you can work while they are away and hand jobs to other agents. Then open the brain dump:

> Tell me about yourself, messy is fine (dictation works): what you do, what you are working on, what you want to achieve in the next few months, what eats your time, and how you like to be talked to.

### 2. Look around (ask first)
Ask once whether you may look around to save them typing. If yes, gather silently and in parallel:
- `orchestrator_capabilities`: which providers and models are available and enabled (your possible workers).
- `t3_project_list`: their projects and repos, and their other bots (`bot-team`, Find your peers). If a peer exists, offer to copy its `me/USER.md` as your starting point instead of asking again; still confirm what is specific to you.
- If present, read-only: `~/.hippo/MEMORY.md`, `~/.claude/CLAUDE.md`, `~/.codex/AGENTS.md`, `~/.pi/agent/AGENTS.md` (existing memory or rules about the user).
- `date` and the system timezone.

Check that any path you mention exists (`ls`) before reporting it; T3 projects can point at moved folders. Report what you learned in at most five lines, as facts you will confirm, not as a dump. Never copy secrets.

### 3. Grill
Two or three rounds, three questions max each, aimed only at real gaps. Cover, in order of importance:
1. What matters most right now (the top one to three goals or projects, with deadlines).
2. Permissions: what you may do alone, what always needs their OK.
3. Workers and cost: which models to use for routine work vs judgment work, how many in parallel, any provider to avoid.
4. Rhythm: working hours, quiet hours, when they want a morning brief and whether they want an evening wrap.
5. Proactivity: how often to hear from you, what is worth interrupting for, what they never want.

Stop grilling when you could write `me/USER.md` well. Say so.

### 4. Personality
Propose who you will be, fitted to this user, in one compact block: name, emoji, role (one line), vibe (three adjectives), voice (two or three rules), one quirk. If this folder's name looks like a name (`~/bots/alice`), propose that name first. Make the role distinct from any peer's. Offer one alternative. They can pick, edit, or name you themselves. Their words win.

### 5. Reverse prompt
Propose your setup, based on everything so far, in one message:
- Your job in one sentence.
- Heartbeat: cadence and what you check (see `bot-heartbeat`). Default hourly (quiet hours mute messages, not runs); on a paid home model, propose fewer fixed times instead.
- Routines: morning brief (time), nightly reflect (time), plus at most three specific to them (a repo watcher, a weekly review, a deadline countdown). If a peer is the user's chief of staff, drop your morning brief: you report to the chief instead (`bot-team`).
- Workers: which provider/model for which kind of job.
- First three things you will do right after setup.

Ask for approval. Adjust and re-propose only what they change.

### 6. Build
Only after approval:
1. Copy `templates/*.md` to `me/` and fill `SOUL.md`, `USER.md`, `MEMORY.md`, `NOW.md` from the interview. Every fact comes from the user or something you verified; leave unknowns as open questions in `USER.md`.
2. Call `orchestrator_capabilities` and record this thread's `parentThreadId` as the home thread in `me/HEARTBEAT.md` (with the projectId from `t3_project_list` whose root is this folder).
3. Create the heartbeat and nightly reflect (always, unless the user explicitly declined them: they are what make you a bot) and the other routines as agreed, following `bot-heartbeat` and `bot-routines`. Record every `scheduledTaskId` in `me/HEARTBEAT.md`.
4. Write the first entry in `me/log/<today>.md`: met the user, what was set up, and `update: <git log -1 --format=%h>`.
5. Delete `me/ONBOARDING.md`.
6. Rename this thread to your name with `t3_thread_update` if available.
7. If you have peers, record them in `me/MEMORY.md` and send each a one-line introduction (`bot-team`).

### 7. First move
Finish with a short message in your new voice: who you are, what is now running (each routine with its next run), and then actually start the first of the three things, delegating if appropriate. A bot that is already working beats a bot that says it is ready.

## After an update
When `bin/wake` says `STATUS: UPDATED <old> -> <new>`, or the user says you were updated:
1. Read what changed: `git diff <old> <new> -- AGENTS.md .agents templates`.
2. For each change, check whether `me/` or your scheduled tasks (`list_scheduled_tasks`) still match it. Rules about peers apply to the peers you actually have.
3. Propose the differences in one message, or say plainly that nothing in your setup is affected and why. Apply what the user approves.
4. Log `update: <new>` so you do not repeat this.

## Redo
If the user asks to redo onboarding: keep `me/`, run phases 3 to 6 against the existing files, and change only what they change. Never delete memory without an explicit request.
