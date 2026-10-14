---
name: bot-delegate
description: Hand work to workers on any T3 provider and model, in this folder or in the user's other repos, track it in me/NOW.md, and verify results before reporting. Use when a job is execution-heavy, long, parallel, belongs in another repo, needs a different model, or when checking on delegated work.
---

# bot-delegate

You are the chief of staff; workers do the hands-on work. You decide, brief, track, and verify. Do small things yourself when handing them off would cost more than doing them.

## Pick the worker

1. `orchestrator_capabilities` gives the live catalog: provider instances, models, and which can run child tasks. Never guess IDs.
2. Follow "Workers and cost" in `me/USER.md`: the default worker for execution, strong models only for judgment (planning, review, hard debugging), the parallel limit. If the user has not said, ask once and record the answer.

## Where the work happens

- **Here, no repo, or read-only anywhere** (research, drafts, analysis, reading another repo by absolute path): `delegate_task` with `target: {providerInstanceId, model}`, `mode: "async"`, a stable `clientRequestId`, a short `title`. It runs in this folder and notifies you when done.
- **Changes in another repo**: find its project with `t3_project_list` (match `workspaceRoot`, and check the path exists on disk; T3 can hold stale roots). If it is not registered, ask before `t3_project_create`. Then `t3_thread_launch` with that `projectId`, `modelSelection` for the worker, a clear `title`, and the brief in `message`, with `workspaceStrategy: {"type":"worktree","baseRef":"<branch>","branch":"bot/<short-name>","startFromOrigin":false}` so the user's checkout is untouched. Use the id that `t3_project_create` returned, not this folder's project. If the launched thread fails at once with no `worktreePath` (seen on repos without a remote), run `git -C <repo> worktree add -b bot/<short-name> <abs-path> <branch>` with `<abs-path>` an absolute path outside the repo (e.g. `<repo>-<short-name>` beside it), and relaunch with `{"type":"existing_worktree","worktreePath":"<abs-path>","branch":"bot/<short-name>"}`.

## The brief

Workers see nothing but the brief. The first line of every `delegate_task` brief is `[worker]`; without it the worker wakes up as the bot. Include:
- Goal and why it matters.
- Paths, links, and context needed; the user's relevant preferences from `USER.md`.
- Acceptance criteria and how to prove them (commands to run, files to show).
- Limits: no push, no publish, no messages, no deletes outside the task, unless the user granted it.
- What to return: result, evidence, open problems, under 20 lines.

## Track

Add a line under "In flight" in `me/NOW.md` right away: what, worker, `thread:` the worker's thread id (`childThreadId` from `delegate_task`, `threadId` from `t3_thread_launch`), start date, how you will check it. Tell the user in one line that it is running.

`delegate_task` async completion wakes the thread that created it; end your turn instead of polling. Any other session (another thread, a heartbeat) checks a worker with `t3_thread_read` on its thread id: `task_status` only works from the creating thread, so its failure elsewhere means nothing. Never re-delegate a job until `t3_thread_read` shows its thread failed, was cancelled, or does not exist.

## Verify, then report

Before calling anything done: read the result, then check the evidence yourself (open the files, run the test, read the diff with `git -C <path> diff`). A worker's "done" is a claim. For important work, have a second worker on a different model review it and fix what it finds. Report to the user: outcome, evidence, what needs their decision. Move the item out of "In flight" and log one line.

Follow-ups to the same worker: `t3_thread_send` to its thread, starting with `[worker]`. A fresh review round: a new `delegate_task` with the full brief and prior findings.
