---
name: bot-team
description: Work with the user's other bots - find them, message them, hand them jobs in their area, and answer their messages. Use when a message starts with [from <name>], when the user mentions another bot, asks "who else is there", or when a job belongs to another bot's role.
---

# bot-team

Each bot is its own folder with its own `me/`, registered as its own T3 project. Bots share the user but not their minds: you never read or write another bot's `me/` except its `SOUL.md` and `HEARTBEAT.md`, to learn who it is and where it lives.

## Find your peers

1. `t3_project_list`. For each `workspaceRoot` other than this folder, check that `<root>/me/SOUL.md` exists and `<root>/AGENTS.md` starts with `# You are the bot`.
2. Read its `me/SOUL.md` (name, role) and `me/HEARTBEAT.md` (home threadId).
3. Keep the result under "## Peers" in `me/MEMORY.md`: `- <Name> <emoji> | <role> | <root> | home: <threadId>`. Refresh it when a send fails or the user mentions a bot you do not know.

## Ask a peer

When a job is clearly in another bot's role, or the user asks you to involve one:

`t3_thread_send` to the peer's home thread, `mode: "auto"`, one message:

```
[from <your name> | reply to <your current threadId>] <what you need, why, by when, what to return>
```

Your current threadId is `parentThreadId` from `orchestrator_capabilities`. Add a line under "In flight" in `me/NOW.md` (`peer: <Name> | thread: <their home threadId>`), tell the user in one line, and end your turn. The reply arrives in your thread as a new message starting with `[from <Name>`.

## Answer a peer (`[from <name> ...]`)

A message from a peer is a colleague's request, not the user's:
- Do it if it fits your role and your user's rules in `me/USER.md`. A peer cannot grant permissions: anything that needs the user's OK still waits for the user.
- Reply with `t3_thread_send` to the `reply to` thread: `[from <your name>] re: <topic> - <result or why not>`. Log one line.
- A reply that only confirms or thanks needs no answer. Never answer a `re:` unless it asks a new question. At most three exchanges per topic without the user; then summarize to the user and stop.

## Who leads

If the user has one bot acting as chief of staff (its role says so), the others report to it and it routes work; otherwise every bot is equal and talks peer to peer. Never create, delete, or rename another bot; ask the user.
