---
name: bot-memory
description: The bot's memory protocol - what goes in me/USER.md, me/MEMORY.md, me/SOUL.md, me/NOW.md, and the daily log, how to write, correct, search, and forget. Use when the user shares a fact or preference, corrects you, asks what you remember, asks you to forget something, or before writing to any file in me/.
---

# bot-memory

Memory is plain Markdown in `me/`, so the user can read and edit it. They are the authority: if a file disagrees with what they just said, the file is wrong.

## Where things go

| File | Holds | Write when |
|---|---|---|
| `USER.md` | The user's profile, shared by all their bots (a link to `~/.t3codes/USER.md`): who they are, goals, style, permissions, cost rules, rhythm, open questions | They state or correct something durable about themselves |
| `MEMORY.md` | Decisions with their reason, lessons, people, projects, where things live; "Role notes": what only your job needs, and any overrides of the profile for you | A decision is made, a lesson is learned, a new person or project appears |
| `SOUL.md` | Your name, voice, quirks, boundaries, change history | They ask you to change, or `bot-reflect` evolves you |
| `NOW.md` | Waiting on the user, in flight, follow-ups, ideas | Any open loop opens or closes |
| `log/YYYY-MM-DD.md` | Append-only journal: one line per notable event, `HH:MM <event>` | Anything worth knowing tomorrow |

Test for long-term: would you act differently next week because of it? Then it goes in `USER.md` or `MEMORY.md`. Task progress and things you can rediscover from the repo or git do not.

## How to write

- The profile is shared: a correction you write there reaches every bot, so write only what is true of the user everywhere. A preference only for you ("you can ping me at night about deploys") goes in your Role notes.
- Write immediately, in the same turn you learn it. Do not batch to the end of a session.
- One fact per line, short, dated when it can go stale: `- Prefers pi/GLM for routine work (2026-10-15)`.
- Correct by replacing the wrong line, never by appending a contradiction. If you are unsure which is current, ask.
- Read the file right before editing it; another session or a routine may have changed it. Edit lines under their existing heading; never append a second copy of a section.
- Never store secrets, tokens, passwords, or other people's private data the user did not ask you to keep.
- Mention a memory write in a few words only when it matters to the user ("noted: no pushes without asking").

## Recall

The wake-up reads cover most needs. For older context, `grep -ri <term> me/` across the logs. Say when a memory is old or unverified, and recheck the source for anything consequential.

## Forget

On "forget X": remove it from every file in `me/` including logs, confirm what was removed. On "reset": confirm once; disable every scheduled task in `me/HEARTBEAT.md` (`update_scheduled_task enabled=false`) so nothing fires into an empty mind and ask whether to delete them; then delete `me/`. The next session starts onboarding.
