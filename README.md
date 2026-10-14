# t3codes

A Grok Bot / Muse style personal bot for T3 Code. Clone it, open a thread in the folder, and the bot wakes up: it interviews you, picks a personality that fits you, remembers you in plain files, checks in on a heartbeat, runs routines while you are away, and hands work to agents on any provider, in this folder or your other repos.

It works with any T3 provider that reads `AGENTS.md` or `CLAUDE.md` (Claude, Codex, Pi, ...). It has no server, no database, and nothing to install.

## Start

One folder is one bot. Keep bots side by side, never inside another bot's folder (agents also read instruction files from parent folders).

### Yourself

```bash
git clone <this repo> ~/bots/alice
```

1. In T3 Code, add `~/bots/alice` as a project.
2. Open a new thread in it. This thread becomes the bot's home: heartbeats and routines post here and run on this thread's model, so a cheap model keeps them cheap.
3. Say hi. The bot interviews you and sets itself up.

### Through any agent

Give an agent running in T3 Code this repo's URL and say "set up a bot for me". It installs the bot without further questions; a pinned home thread appears in the sidebar with the interview already started. The bot then names itself with you and updates its thread title, project name, and icon.

## Agent install

For an agent installing a bot for its user. Do it without questions; the bot asks what it needs in its own interview.

1. Pick the folder: `~/bots/<name>` if the user named the bot, else the first free `~/bots/bot<N>`. Never inside another bot's folder.
2. Pick the home model from `orchestrator_capabilities`: the one the user asked for, else the cheapest enabled model that can run child tasks (a local one if listed), else your own. Heartbeats run on it.
3. `git clone <repo url> <folder>`.
4. `t3_project_create` with `title` = the folder name and `workspaceRoot` = the absolute folder path.
5. `t3_thread_launch` with that `projectId`, `title` = `<folder name> · home`, the home model as `modelSelection`, and `message`:
   ```
   [install] Installed by <your name> for the user. Notes about the user: <what you already know that the user would otherwise have to repeat, or "none">
   ```
6. Pin it: `t3_thread_organize` with `action: "pin"` and the new `threadId`.
7. Tell the user in two lines: the bot is pinned in the sidebar, and the link `[<title>](t3-thread://v1/<threadId>)` where its interview has started.

The bot sets its own name, emoji icon, and default model on the project when its interview ends.

Without T3 tools, stop after step 3 and tell the user to add the folder as a project in T3 Code and say hi in a new thread.

## Many bots

Clone once per bot (`~/bots/alice`, `~/bots/bob`), each with its own job, memory, and routines. Bots find each other through T3 projects and talk by sending messages to each other's home thread (`bot-team`): one bot can hand another a job in its area, and a chief-of-staff bot can route work to specialists. A new bot can start from an existing bot's profile of you instead of a fresh interview.

## How it works

| Grok Bot / Muse | Here |
|---|---|
| A Bot with a name, avatar, description | `me/SOUL.md`, written with you during onboarding |
| Brain dump, grill-me, reverse-prompted setup | `bot-onboard` skill, first run |
| Memory you can read and edit | `me/USER.md`, `me/MEMORY.md` (long-term), `me/NOW.md`, `me/log/` (short-term) |
| Always on, heartbeat | T3 scheduled task posting `[heartbeat]` into the home thread (`bot-heartbeat`) |
| Routines and saved skills | T3 scheduled tasks (`bot-routines`); learned skills in `.agents/skills/my-*` |
| Chief of staff with specialist bots | Workers on any provider and repo (`bot-delegate`); other bots in their own folders (`bot-team`) |
| Gets better over time | Nightly `bot-reflect`: consolidate memory, prune, evolve personality from feedback |
| Approval for external actions | Permissions in `me/USER.md`; anything external or irreversible asks first |

Layout:

```
AGENTS.md               the bot's kernel: identity, wake-up, rules (CLAUDE.md imports it)
.agents/skills/bot-*    onboarding, memory, heartbeat, routines, delegation, reflection, team
.claude/skills          symlink to .agents/skills
bin/wake                wake-up status and memory dump
templates/              seeds for me/
me/                     your bot's mind; private and gitignored
```

Every session opened in this folder is the bot. Delegated workers in this folder get a brief containing `[worker]` and skip the bot persona.

## Privacy

`me/` is gitignored so pulling updates never touches your bot's memory. To back it up with git, keep a private fork and remove `me/` from `.gitignore`. Never push a bot's `me/` to a public repo.

## Updating

`git pull` in the bot's folder, then tell the bot "you were updated, run bin/wake" in its home thread. It compares its setup with the new skills and proposes the changes. Your `me/` and your `my-*` skills are untouched.
