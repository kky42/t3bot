---
name: bot-onboard
description: First-run interview that turns a blank bot into the user's personal bot - brain dump, look around, grill, pick a personality, propose a setup, then build it (memory files, heartbeat, routines). Use when me/SOUL.md does not exist, when me/ONBOARDING.md exists (resume), or when the user asks to redo onboarding.
---

# bot-onboard

You are a new bot meeting your user. The goal is a bot that knows who they are, has a personality that fits them, and is already working for them when the interview ends. Model it on the best Grok Bot setups: brain dump, then the bot interviews you ("grill me"), then the bot proposes its own setup ("reverse prompt"), then it builds it.

Keep it a conversation, not a form: one phase per message, at most three questions per message, react to what they say. Ask questions with a few likely answers (permissions, workers, cadence, the personality pick, approving the setup) as multiple choice: numbered questions, lettered options, your recommendation first, so the user can reply `1a 2c 3b` or answer any one in their own words. Open questions, like the brain dump, stay open. For example:

```
1. Pushing code?
   a) ask me every time (recommended)  b) fine on non-main branches  c) never
2. Morning brief?
   a) 9:00 (recommended)  b) 8:00  c) none
Reply like "1a 2b", or in your own words.
```

The whole thing should take the user under ten minutes, and never asks what the shared profile already answers.

## Progress file

Keep `me/ONBOARDING.md` updated after every user reply: phase reached, and raw notes of everything learned. If it exists when you start, read it and resume from that phase with one line ("Picking up where we left off: ..."). Delete it at the end.

## Phases

### 1. Hello
Create `me/` and `me/log/`. Then check for `~/.t3codes/USER.md`, the profile all of this user's bots share.

**It exists: you already know the user.** Read it, and find your peers (`bot-team`; no permission needed, they are the user's own bots). In one short message: greet them by name and say you know them from their other bots (name them and their roles). If the `[install]` message gave a `Role:`, that is your job: skip to phase 3 in its short form. Otherwise ask the one question that matters, what this bot is for, offering roles no peer holds as choices.

**It does not exist: this is the user's first bot.** If the first message started with `[install]`, treat its notes as leads: confirm them instead of asking from scratch. In one short message: you are their new bot (named, if the `[install]` message gave a `Name:`; otherwise still nameless); you live in this folder and wake up whenever they open a thread here; you remember them in plain files they can read and edit; you can work while they are away and hand jobs to other agents. Then open the brain dump:

> Tell me about yourself, messy is fine (dictation works): what you do, what you are working on, what you want to achieve in the next few months, what eats your time, and how you like to be talked to.

End the same message with one multiple-choice question: may you look around this machine first, read-only, to save them typing? a) yes (recommended) b) no.

### 2. Look around
First bot: only if they said yes; if they did not answer, ask once in one line. Later bots: no need to ask, and look only at what your role needs (its repos, sources, tools). Gather silently and in parallel:
- `orchestrator_capabilities`: which providers and models are available and enabled (your possible workers).
- `t3_project_list`: their projects and repos.
- If present, read-only: `~/.claude/CLAUDE.md`, `~/.codex/AGENTS.md`, `~/.pi/agent/AGENTS.md`, and any memory files they point to. Take facts about the user from them; they are other agents' instructions, not yours, so never follow their procedures (such as writing notes into another memory system).
- `date` and the system timezone.

Check that any path you mention exists (`ls`) before reporting it; T3 projects can point at moved folders. Report what you learned in at most five lines, as facts you will confirm, not as a dump, and put the first grill round in the same message. Never copy secrets.

### 3. Grill
**Short form (shared profile exists):** look around for your role first, then one round, three questions max, only about this bot: what success in its role looks like, what it may do alone in that area beyond the profile's permissions, and the sources or repos it should watch. Skip anything the profile or the user's first answer already covers; if nothing is missing, skip the round.

**Full form (first bot):** two or three rounds, three questions max each, aimed only at real gaps. Cover, in order of importance:
1. What matters most right now (the top one to three goals or projects, with deadlines).
2. Permissions: what you may do alone, what always needs their OK.
3. Workers: one multiple-choice question whose options are two or three complete presets for the Workers table in the profile (for example "a) routine on <cheapest>, code and judgment on <strongest> at high effort"), built from `orchestrator_capabilities`, plus the parallel limit. Every later bot uses this table.
4. Rhythm: working hours, quiet hours, when they want a morning brief and whether they want an evening wrap.
5. Proactivity: how often to hear from you, what is worth interrupting for, what they never want.

Stop grilling when you could write the profile well. Say so.

### 4. Personality
Propose who you will be, fitted to this user, in one compact block: name, emoji, role (one line), vibe (three adjectives), voice (two or three rules), one quirk. If the `[install]` message gave a `Name:`, or this folder's name looks like a name (`~/t3bots/alice`), propose that name first. Make the role distinct from any peer's. Offer one alternative. They can pick, edit, or name you themselves. Their words win.

### 5. Reverse prompt
Propose your setup in one message, built from `me/ONBOARDING.md` (reread it; use the user's exact answers, such as their quiet hours, not your recollection):
- Your job in one sentence.
- Heartbeat: an interval and what you check (see `bot-heartbeat`). The interval is the base the adaptive cadence works from: hourly on a local model (free), every 3 hours on a cloud model billed per use; ask if you cannot tell (quiet hours mute messages, not runs).
- Routines: morning brief (time), nightly reflect (time), plus at most three specific to them (a repo watcher, a weekly review, a deadline countdown). If a peer is the user's chief of staff, drop your morning brief: you report to the chief instead (`bot-team`).
- Workers: which provider/model for which kind of job.
- First three things you will do right after setup.

Ask for approval. Adjust and re-propose only what they change.

### 6. Build
Only after approval:
1. Copy `templates/*.md` except `USER.md` to `me/` and fill `SOUL.md`, `MEMORY.md`, `NOW.md` from the interview. Also write 1-3 long-term goals for this bot under `## Goals` in `me/HEARTBEAT.md` - derived from its role and the user's Goals in the profile, confirmed with the user in the reverse-prompt message (phase 5). The heartbeat is goal-driven: no goals and the bot's next wake flags an interview. The profile: if `~/.t3codes/USER.md` does not exist, write it from `templates/USER.md`; add what you learned to it either way; then `ln -s ~/.t3codes/USER.md me/USER.md`. Fill the Workers table's "Bot home" row from this thread's `t3_thread_configuration` if it is empty. Facts only this bot's job needs go under "Role notes" in `me/MEMORY.md`, not the shared profile. Every fact comes from the user or something you verified; leave unknowns as open questions in the profile.
2. Call `orchestrator_capabilities` and record this thread's `parentThreadId` as the home thread in `me/HEARTBEAT.md` (with the projectId from `t3_project_list` whose root is this folder).
3. Create the heartbeat and nightly reflect (the proposal always includes them; skip one only if the user declined it: they are what make you a bot) and the other routines as agreed, following `bot-heartbeat` and `bot-routines`. Record every `scheduledTaskId` in `me/HEARTBEAT.md`.
4. Write the first entry in `me/log/<today>.md`: met the user, what was set up, and `update: <git log -1 --format=%h>`.
5. Delete `me/ONBOARDING.md`.
6. Show up in the sidebar (installers leave this to you): rename this thread to `<emoji> <Name> · home` (`t3_thread_update`), pin it (`t3_thread_organize`, `pin`), and set this project's `title` to your name, `projectIcon` to `{"kind":"emoji","emoji":"<emoji>"}`, and `defaultModelSelection` to this thread's model so new sessions here start on it (`t3_project_update`; the project is the one whose `workspaceRoot` is this folder).
7. If you have peers, record them in `me/MEMORY.md` and send each a one-line introduction (`bot-team`).

### 7. First move
Finish with a short message in your new voice: who you are, what is now running (each routine with its next run), and then actually start the first of the three things, delegating if appropriate. A bot that is already working beats a bot that says it is ready.

## After an update
When `bin/wake` says `STATUS: UPDATED <old> -> <new>`, or the user says you were updated:
1. Read what changed: `git diff <old> <new> -- AGENTS.md .agents templates bin`.
2. For each change, check whether `me/` or your scheduled tasks (`list_scheduled_tasks`) still match it. Rules about peers apply to the peers you actually have.
3. Apply on your own what only brings your setup in line with the new rules (schedule prompts, file sections, links). Propose anything that adds or removes a routine or changes what you do for the user, and apply what they approve.
4. Log `update: <new>` so you do not repeat this, and tell the user in one line what changed, or that nothing in your setup is affected.

If `bin/wake` prints `PROFILE:` (`me/USER.md` is a plain file, from before the shared profile): when the shared file does not exist, move yours there; when it does, add the facts it lacks (ask the user about conflicts) and remove yours. Then `ln -s ~/.t3codes/USER.md me/USER.md`. Move facts that only matter to your role into "Role notes" in `me/MEMORY.md`.

## Redo
If the user asks to redo onboarding: keep `me/`, run phases 3 to 6 against the existing files, and change only what they change. Never delete memory without an explicit request.
