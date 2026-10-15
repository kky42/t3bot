# t3codes

A Grok Bot / Muse style personal bot for T3 Code. Clone it, open a thread in the folder, and the bot wakes up: it interviews you, picks a personality that fits you, remembers you in plain files, checks in on a heartbeat, runs routines while you are away, and hands work to agents on any provider, in this folder or your other repos.

It works with any T3 provider that reads `AGENTS.md` or `CLAUDE.md` (Claude, Codex, Pi, ...). It has no server, no database, and nothing to install.

## Start

One folder is one bot. Keep bots side by side, never inside another bot's folder (agents also read instruction files from parent folders).

### Yourself

```bash
git clone https://github.com/kky42/t3bot ~/t3bots/alice
```

1. In T3 Code, add `~/t3bots/alice` as a project.
2. Open a new thread in it. This thread becomes the bot's home: heartbeats and routines post here and run on this thread's model, so a cheap model keeps them cheap.
3. Say hi. The bot interviews you and sets itself up.

### Through any agent

Give an agent running in T3 Code this repo's URL and say "set up a bot for me". It asks you for the bot's name, offering a few suggestions, then installs it in `~/t3bots/<name>`; a pinned home thread appears in the sidebar with the interview already started. The bot then sets its emoji icon and updates its thread title and project.

## Agent install

For an agent installing a bot for its user. Ask only for the name; the bot asks everything else in its own interview.

1. Name: if the user has not named the bot, ask them. Offer three or four short, fun, distinct names that fit what you know of them as choices, say any other name works too, and wait for the reply.
2. Folder: `~/t3bots/<slug>`, where `<slug>` is the name in lowercase with spaces as hyphens and only `a-z`, `0-9`, `-` kept. If that folder exists, ask for another name.
3. Pick the home model from `orchestrator_capabilities`: the one the user asked for, else the cheapest enabled model that can run child tasks (a local one if listed), else your own. Heartbeats run on it.
4. `git clone <repo url> <folder>`.
5. `t3_project_create` with `title` = the name and `workspaceRoot` = the absolute folder path.
6. `t3_thread_launch` with that `projectId`, `title` = `<name> · home`, the home model as `modelSelection`, and `message`:
   ```
   [install] Installed by <your name> for the user. Name: <name>. Notes about the user: <what you already know that the user would otherwise have to repeat, or "none">
   ```
7. Pin it: `t3_thread_organize` with `action: "pin"` and the new `threadId`.
8. Tell the user in two lines: the bot is pinned in the sidebar, and the link `[<title>](t3-thread://v1/<threadId>)` where its interview has started.

The bot sets its emoji icon and default model on the project when its interview ends.

Without T3 tools, stop after step 4 and tell the user to add the folder as a project in T3 Code and say hi in a new thread.

## Many bots

Clone once per bot (`~/t3bots/alice`, `~/t3bots/bob`), each with its own job, memory, and routines. Bots find each other through T3 projects and talk by sending messages to each other's home thread (`bot-team`): one bot can hand another a job in its area, and a chief-of-staff bot can route work to specialists. Your profile lives once, in `~/.t3codes/USER.md`, linked from every bot: the first bot interviews you, later bots only ask what their own job needs, and a correction told to any bot reaches all of them.

## How it works

| Grok Bot / Muse | Here |
|---|---|
| A Bot with a name, avatar, description | `me/SOUL.md`, written with you during onboarding |
| Brain dump, grill-me, reverse-prompted setup | `bot-onboard` skill, first run |
| Memory you can read and edit | `~/.t3codes/USER.md` (your profile, shared by all your bots), `me/MEMORY.md` (long-term), `me/NOW.md`, `me/log/` (short-term) |
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
bin/update              reset to upstream, keeping me/ and my-* skills (run nightly by reflect)
templates/              seeds for me/
me/                     your bot's mind; private and gitignored
```

Every session opened in this folder is the bot. Delegated workers in this folder get a brief containing `[worker]` and skip the bot persona.

## Privacy

`me/` is gitignored so pulling updates never touches your bot's memory, and your profile lives outside every repo in `~/.t3codes/`. To back it up with git, keep a private fork and remove `me/` from `.gitignore`. Never push a bot's `me/` to a public repo.

## Updating

Bots update themselves: each nightly reflect runs `bin/update`, which resets the folder to the upstream branch, and the bot adapts its setup to the new rules and tells you in one line what changed. Changes to what it does for you, such as adding or removing a routine, still wait for your OK. Edits to tracked files and local commits are discarded, so customize a bot only in `me/` and `my-*` skills, or in your own fork. To update now, run `bin/update` in the bot's folder and tell the bot "you were updated, run bin/wake". To turn auto-updates off, tell the bot. Your `me/` and your `my-*` skills are untouched.
