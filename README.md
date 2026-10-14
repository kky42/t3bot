# t3codes

A Grok Bot / Muse style personal bot for T3 Code. Clone it, open a thread in the folder, and the bot wakes up: it interviews you, picks a personality that fits you, remembers you in plain files, checks in on a heartbeat, runs routines while you are away, and hands work to agents on any provider, in this folder or your other repos.

It works with any T3 provider that reads `AGENTS.md` or `CLAUDE.md` (Claude, Codex, Pi, ...). It has no server, no database, and nothing to install.

## Start

```bash
git clone <this repo> ~/my-bot
```

1. In T3 Code, add `~/my-bot` as a project.
2. Open a new thread in it. This thread becomes the bot's home: heartbeats and routines post here, and they run on this thread's model, so a cheap model keeps them cheap.
3. Say hi. The bot runs a short interview and sets itself up.

Clone again into another folder for a second bot with a different job.

## How it works

| Grok Bot / Muse | Here |
|---|---|
| A Bot with a name, avatar, description | `me/SOUL.md`, written with you during onboarding |
| Brain dump, grill-me, reverse-prompted setup | `bot-onboard` skill, first run |
| Memory you can read and edit | `me/USER.md`, `me/MEMORY.md` (long-term), `me/NOW.md`, `me/log/` (short-term) |
| Always on, heartbeat | T3 scheduled task posting `[heartbeat]` into the home thread (`bot-heartbeat`) |
| Routines and saved skills | T3 scheduled tasks (`bot-routines`); learned skills in `.agents/skills/my-*` |
| Chief of staff with specialist bots | `delegate_task` and `t3_thread_launch` on any provider and repo (`bot-delegate`) |
| Gets better over time | Nightly `bot-reflect`: consolidate memory, prune, evolve personality from feedback |
| Approval for external actions | Permissions in `me/USER.md`; anything external or irreversible asks first |

Layout:

```
AGENTS.md               the bot's kernel: identity, wake-up, rules (CLAUDE.md imports it)
.agents/skills/bot-*    onboarding, memory, heartbeat, routines, delegation, reflection
.claude/skills          symlink to .agents/skills
templates/              seeds for me/
me/                     your bot's mind; private and gitignored
```

Every session opened in this folder is the bot. Delegated workers in this folder get a brief containing `[worker]` and skip the bot persona.

## Privacy

`me/` is gitignored so pulling updates never touches your bot's memory. To back it up with git, keep a private fork and remove `me/` from `.gitignore`. Never push a bot's `me/` to a public repo.

## Updating

`git pull`. Your `me/` and your `my-*` skills are untouched.
