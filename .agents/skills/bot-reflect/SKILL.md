---
name: bot-reflect
description: Nightly consolidation and self-evolution - fold the day's log into long-term memory, prune NOW.md, check routine health, adjust personality from feedback, and prepare tomorrow. Use on a [routine] reflect run, or when the user asks you to reflect, tidy your memory, or change who you are.
---

# bot-reflect

Sleep for a bot: turn the day into memory, keep the files small and true, and get a little better at being this user's bot.

## Steps

1. **Read** `me/log/` for today (and any day since the last reflect, found by its `reflect` log line), plus `NOW.md`, `USER.md`, `MEMORY.md`, `SOUL.md`.
2. **Promote**: durable facts and preferences to `USER.md`, decisions and lessons to `MEMORY.md`, following `bot-memory`. Replace stale lines; merge duplicates. Keep `MEMORY.md` under about 200 lines by compressing old entries.
3. **Prune** `NOW.md`: drop finished items, flag anything in flight for over two days, move overdue follow-ups to "Waiting on the user".
4. **Routines health**: compare `list_scheduled_tasks` with the Routines list in `me/HEARTBEAT.md`. Fix the file; report any task that is missing, disabled without reason, or whose last run failed. Fix any heartbeat or routine prompt that drifted from the form in `bot-heartbeat` and `bot-routines` (`update_scheduled_task`).
5. **Feedback**: look for signs in the log of what worked and what annoyed the user (corrections, ignored messages, "too long", "stop doing X", thanks). Turn each into a concrete rule in `USER.md` or `SOUL.md`. If heartbeats are mostly noise, propose a slower cadence; if the user keeps asking for something, propose a routine.
6. **Evolve**: personality changes are small and earned by feedback, never random. Record each under "Changes" in `SOUL.md` with date and reason, and tell the user about it in the next message they will read. Never change your name or core role without asking.
7. **Learn**: if `USER.md` has open questions and none was asked today, put the best one under "Waiting on the user" for the morning brief.
8. **Tomorrow**: refresh "Ideas" in `NOW.md`: up to three things you could do for the user, phrased "I can ...", each tied to a goal in `USER.md`.
9. **Report up**: if a peer is the user's chief of staff and you are not, send it your status (`bot-team`, Who leads).
10. **Log** `HH:MM reflect: <one line of what changed>`.

Reply `ROUTINE_OK reflect` and nothing else unless step 4 or 6 produced something the user should know; then say it in two lines max.
