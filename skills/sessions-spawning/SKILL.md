---
name: sessions-spawning
description: >
  How to spawn OpenCode worker sessions and tasks, pick their models through
  the model-routing skill, and exchange messages with them. Load before
  spawning any session or task to delegate work.
---

# Spawning sessions and tasks

A "worker" is a session or a task. Use one to keep this context window small, or to run a job on a cheaper or more capable model.

- Tasks for short, blocking work; sessions for long work, work that spawns other workers, or work running in parallel with you.
- Reuse a warm worker that already has the checkout for micro-fixes and follow-ups on its own work.
- A Manager session is allowed for very complex work. The user owns it, so never reuse a Manager; messaging one with news or a progress question is fine.

## Local conventions

Defaults; values in the global or project `AGENTS.md` take precedence.

- Message directory: `${TMPDIR:-/tmp}/opencode/agent-messages`. Every session that reads or writes messages must see it at the same absolute path. A reboot clears it, so when sessions outlive reboots `AGENTS.md` should set a persistent one.
- Session titles: `W: ` for a worker, `M: ` for a manager, `(DONE) ` once finished.

## Tools

- Tasks: the built-in `task`, passing `model` and `variant`. If its output starts with a `task:` warning that they did not apply, use `task_with_model` (plugin `opencode-task-with-model`), which always runs on the model you name. Without it, stop and tell the user; never let a worker run on a model you didn't pick.
- Sessions need `spawn_session`, `send_agent_message` and `reply` (plugin `marenz/opencode-plugins`). Without them, use tasks or ask the user.

## Choosing the model

- ALWAYS load the `model-routing` skill before choosing a model, and follow it. For each worker it gives a `provider/model`, an effort, whether the route is heavy with the provider's limit on heavy workers, and whether it is for bounded work only; or no suitable route.
- Pass the model as `model` and the effort as `variant` to `task`, `task_with_model` or `spawn_session`; without `variant` the worker runs at its agent's default effort.
- A route for bounded work only takes one bounded job, never a loop or a long-running session.
- Second-opinion reviews go to a different model than the author's.
- When routing fails or finds no route, fix the cause or ask the user; never pick a model some other way.
- Only when no `model-routing` skill exists: pick from `list_models` (and the `quota` tool, if installed), prefer the more capable model when in doubt, never use a provider or model that trains on submitted data or keeps it beyond short operational retention, and treat every `xhigh` or `max` worker as heavy, at most one per provider.

## Setup

- One worker per disjoint scope: separate repos/worktrees, never two writers in one checkout.
- Prefix a spawned session's title (NOT a task's) with the worker prefix, or the manager prefix if it is itself a manager. Reusing a warm session, drop its finished prefix and keep the worker prefix.
- ASK before reusing a session without the worker prefix: it may be user-owned.
- Delegations and final reports follow "Inter-agent messages" below.
- A heavy worker is long-running, or on a route marked heavy. Keep under the provider's limit on heavy workers.
- Workers may start workers of their own unless their delegation says otherwise. Lightweight agents (`explore`, `librarian`) and short, bounded tasks on routes not marked heavy are fine. Be careful with heavy ones and mention each to your spawner (agent, model, variant, why): a session worker with a one-line `reply` when starting it, a task in its final answer.
- As a spawner, keep count of everything running under you, your workers' workers included, and tell a worker to slow down when it starts heavy workers too often.

## Inter-agent messages

Files keep unread detail out of the context window and long messages readable for the user.

- Store substantial or heavily formatted messages in `<message directory>/YYYY-MM-DD/YYYYMMDDTHHMMSSZ-NN__from-SRC__to-DST.md`. Use UTC, complete session IDs, and a two-digit counter from `01`. The filename stem is the message ID. NEVER overwrite, rename or delete a message; increment the counter on collision. A delegation written before its worker exists has no recipient ID: put the literal `__new-session` where `__to-DST` goes, and the worker's reply links back with `In reply to:`.
- A reply file starts with `In reply to: <parent-stem>`, omitted for a new thread. Find a parent with `ls <message directory>/*/<parent-stem>.md`.
- Files otherwise contain ONLY descriptive `##` body sections: subject, summary and section index stay in band, never duplicated in the file. Message files are immutable after sending.
- MUST use a file for delegations, final reports, messages over about 15 lines, and anything with logs, diffs or tables. Very short acknowledgements and one-line questions may stay inline. A worker whose permissions or delegation forbid writing files reports inline instead, and its spawner accepts that.
- Messaging a running session, send only this compact notice through `send_agent_message` or `reply`: `Subject`, 1-3 line `Summary`, absolute `File` path, useful indexed `Sections`. Index entries use inclusive line ranges, as `L<start>-L<end> <heading>: <read-this-when hint>`. No body details, and NEVER paste file contents in band.
- A new worker always reads its complete delegation, so its `spawn_session` prompt MUST be only `Read the full delegation: <absolute-file-path>`: no Subject, Summary or Sections index, which exist for a running session that may skip parts.
- On receipt, read the in-band summary first, and nothing else when that is enough to decide. Otherwise use the listed line ranges with `Read`'s `offset`/`limit`; `rg -n '^## ' <file>` only if the index is missing or suspect. Record which sections you read only when it affects confidence.
- A worker MUST deliver its final report through `reply` before its last turn.
- A task's final reply is its final answer, not a `reply` or `send_agent_message`; a long report goes in a file and the answer takes the shape of a notice.

Format example, notice then referenced file:

```text
Subject: Retry API cannot preserve cancellation
Summary: The proposed retry path is unsafe because it retries cancellation.
Action: discard this option unless the API adds a cancellation exception.
File: /tmp/opencode/agent-messages/2026-09-11/20260911T143112Z-01__from-ses_worker__to-ses_manager.md
Sections:
- L1-L4 Verdict: read to decide whether to discard the option.
- L5-L9 Evidence: read to validate the API behavior.
```

```markdown
## Verdict

The retry API cannot preserve cancellation with the proposed design.

## Evidence

Cancellation and transport errors share the same retry branch.
```

## Cleanup

- Prepend the finished prefix to a session's title once its work is verified correct and finished.
- Remove stale worktrees and branches when the work is merged or abandoned.
