---
name: bot-orchestrate
description: Run work through workers on any T3 provider and model, from one quick job to a multi-day program - pick the worker, write the brief, track it in me/NOW.md, verify, and report. Use when a job is execution-heavy, long, parallel, belongs in another repo, needs a different model, or when checking on work in flight.
---

# bot-orchestrate

You own outcomes, not keystrokes: you decide, brief, track, verify, and report; workers do the hands-on work. Do it yourself when briefing would cost more than doing.

## Rules

- **Never block on the user for reversible work.** Proceed, show the result, let them correct it. If the answer is a fact you could check (run it, read it, try it), check it instead of asking. Ask only for a preference no experiment settles, or before anything their permissions in `me/USER.md` gate. Give your real judgment, including "this is not worth doing".
- **The brief is the product.** A worker sees nothing but the brief and cannot ask you anything; a vague brief fails quietly.
- **Completions are events.** Record them and keep going; never sit polling a worker.
- **Guard your context.** Send bulk reading and searching to workers; keep their summaries, not their raw output.
- **Encode, don't repeat.** When you restate an instruction to a worker or get the same correction twice, add a line to `me/USER.md` or `me/MEMORY.md` so every future brief carries it.

## Pick the worker

`orchestrator_capabilities` is the live catalog; never guess ids. "Workers" in `me/USER.md` maps each kind of work to a provider, model, and effort, and sets the parallel limit; follow it, and fill a missing row by asking once. Then tune effort to the job: a step lower for routine work, a step higher for the hardest. Set effort as `options: [{"id": "<the model's reasoning option id>", "value": "<level>"}]` in the model selection. A second opinion or a check goes to a different model family than the work it checks.

## Where the work happens

- **Here, no repo, or read-only anywhere** (research, drafts, analysis, reading another repo by absolute path): `delegate_task` with `target: {providerInstanceId, model, options}`, a `role` (T3 then marks the brief as a sub-agent's), `mode: "async"`, a stable `clientRequestId`, a short `title`. It runs in this folder and notifies you when done.
- **Changes in another repo**: find its project with `t3_project_list` (match `workspaceRoot`, and check the path exists on disk; T3 can hold stale roots). If it is not registered, ask before `t3_project_create`. Then `t3_thread_launch` with that `projectId`, `modelSelection` for the worker, a clear `title`, and the brief in `message`, with `workspaceStrategy: {"type":"worktree","baseRef":"<branch>","branch":"bot/<short-name>","startFromOrigin":false}` so the user's checkout is untouched. If the thread fails at once with no `worktreePath` (seen on repos without a remote), run `git -C <repo> worktree add -b bot/<short-name> <abs-path> <branch>` with `<abs-path>` outside the repo, and relaunch with `{"type":"existing_worktree","worktreePath":"<abs-path>","branch":"bot/<short-name>"}`. One writer per branch or folder.

## The brief

First line `[worker]`; without it the worker wakes up as the bot. Then, sized to the job (a one-command job gets one paragraph that still covers goal, check, and report):

```
GOAL        one sentence a stranger could execute, and why it matters
CONTEXT     paths and links; earlier workers' results pasted in full when this job builds on them
SCOPE       what it may change; what it must not touch
DONE WHEN   checkable criteria, one per line
CHECK       the exact commands or steps that prove it
TIMEBOX     rough cap; on expiry, return partial findings and stop
LIMITS      no push, publish, message, purchase, or delete beyond scope unless USER.md grants it
REPORT      result, what it actually ran, evidence, deviations, open problems; under 20 lines
STANDING    the lines of USER.md and MEMORY.md this job touches (preferences, bans)
```

A field you cannot fill means the job is not scoped yet. New work, a fix round, or a retry goes to a fresh worker with consolidated scope: the brief, every later instruction, and the previous report. Message an existing worker (`t3_thread_send`, starting `[worker]`) only when the next step needs state that lives there, like its uncommitted changes.

## Track

Add a line under "In flight" in `me/NOW.md` right away: what, worker, `thread:` its thread id (`childThreadId` from `delegate_task`, never the `taskId`; `threadId` from `t3_thread_launch`), start date, how you will check it. Tell the user in one line that it is running, then end your turn: async completion wakes the thread that started it.

Any other session (another thread, a heartbeat) checks a worker with `t3_thread_read` on its thread id; `task_status` only works from the creating thread. `t3_thread_list` without `projectId` lists only this project; pass each project's id to see the others. Never re-delegate until `t3_thread_read` shows the thread failed, was cancelled, or does not exist. Retry by cause: too big, split it; a tool or model error, another model; unknown, once. After two failed retries, drop it and tell the user what you tried.

## Bigger programs

When the work outlives one worker (many steps, many repos, days):
1. State the done condition as something countable ("all 12 repos have a LICENSE, each checked").
2. Cut it into units that each end in a check. Run the first unit alone as a pilot and fix the brief from what it shows before fanning out.
3. Keep a rolling window of workers up to the parallel limit, refilling as each finishes; pass a finished unit's result into the briefs that depend on it.
4. Track every unit in `NOW.md`; land and record each as it passes, never in a batch at the end.
5. Park every decision that needs the user under "Waiting on the user" with your default, keep working around it, and ask them all at once. Retries, routine fixes, and "should I keep going" never reach the user.

## Verify, then report

A worker's "done" is a claim. Scale the check to the stakes: rerun its check command and open the evidence yourself (files, diff, output); for important work, a second worker on another model family reviews it and fixes what it finds. Write "verified" only about what you checked yourself, and say how; label the rest as the worker's findings. Report outcome first, every claim with its evidence or labeled as inferred or a guess, then what needs the user's decision. Never hand the user a check you could run. Move the item out of "In flight" and log one line.
