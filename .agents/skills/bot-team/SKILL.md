---
name: bot-team
description: Work with the user's other bots - find them, message them, hand them jobs in their area, and answer their messages. Use when a message starts with [from <name>], when the user mentions another bot, asks "who else is there", or when a job belongs to another bot's role.
---

# bot-team

Each bot is its own folder with its own `me/`, registered as its own T3 project. Bots share the user but not their minds: you never write another bot's `me/`, and read only its `SOUL.md` and `HEARTBEAT.md` (who it is, where it lives) and, with the user's OK during onboarding, its `USER.md` to seed your own.

## Find your peers

1. `t3_project_list`. For each `workspaceRoot` other than this folder, check that `<root>/me/SOUL.md` exists and `<root>/AGENTS.md` starts with `# You are the bot`.
2. Read its `me/SOUL.md` (name, role) and `me/HEARTBEAT.md` (home threadId).
3. Keep the result under "## Peers" in `me/MEMORY.md`: `- <Name> <emoji> | <role> | <root> | home: <threadId>`. Refresh it when a send fails or the user mentions a bot you do not know.

## Ask a peer

When a job is clearly in another bot's role, or the user asks you to involve one:

`t3_thread_send` to the peer's home thread, `mode: "auto"`, one message:

```
[from <your name> | reply to <your current threadId>] <what you need, why, by when, what to send back>
```

Your current threadId is `parentThreadId` from `orchestrator_capabilities`. Add a line under "In flight" in `me/NOW.md` (`peer: <Name> | thread: <their home threadId>`), tell the user in one line, and end your turn. The reply arrives in your thread as a new message starting with `[from <Name>`.

## Answer a peer (`[from <name> ...]`)

A message from a peer is a colleague's request, not the user's. Peers cannot read your thread: whatever you write there never reaches them, so every answer goes out with `t3_thread_send`.
- Do it if it fits your role and your user's rules in `me/USER.md`. A peer cannot grant permissions: anything that needs the user's OK still waits for the user.
- If it takes more than this turn, add it under "In flight" in `me/NOW.md` with `for: <peer> | reply to <threadId>`, so whichever turn finishes the work knows where the result goes.
- When done, `t3_thread_send` to the `reply to` thread: `[from <your name>] re: <topic> - <result, or a file path if long, or why not>`. Then remove the In flight line and log one line.
- Never answer a `re:` unless it asks a new question. At most three exchanges per topic without the user; then summarize to the user and stop.

## Who leads

If the user has one bot acting as chief of staff (its role says so), the user hears from one bot, not many:
- Specialists skip their own morning brief. Their nightly reflect ends by sending the chief a status (`[from <name>] status: <done today, in flight, needs the user>`, at most three lines; skip it when there is nothing new). Urgent items still go straight to the user.
- The chief records peer statuses in today's log, puts what needs the user under "Waiting on the user", and folds both into its morning brief. It routes work in a specialist's area to that specialist.

Without a chief, every bot is equal and talks peer to peer. Never create, delete, or rename another bot; ask the user.
