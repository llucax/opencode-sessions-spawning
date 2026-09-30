---
name: sessions-spawning
description: >
  How to spawn OpenCode worker sessions and tasks and keep track of them.
  Load before spawning a session, or before a task when you manage several
  workers.
---

# Spawning sessions and tasks

A worker is a session or a task.

- Tasks for short, blocking work. Sessions for long work, parallel work, or work that spawns workers of its own.
- Reuse a warm worker for follow-ups on its own work.
- Load `agent-messages` to write delegations and read reports.

## Model

- Call the `model_route` tool with the `job` that fits, or the `job` or `score` and `tags` the agent's description recommends; add `not_model` with the author's model for second-opinion reviews. Its first line is the route: a `provider/model`, an effort and its notes; or it says no route fits.
- Pass the model as `model` and the effort as `variant` to `task`, `task_with_model` or `spawn_session`.
- Respect the notes. `heavy`: the provider's limit on heavy workers at once; long-running workers count as heavy too. `bounded work only`: one bounded job, never a loop or a long-running session.
- When routing fails or finds no route, fix the cause or ask the user; never pick a model another way.
- Only if there is no `model_route` tool: pick from `list_models` (and the `quota` tool, if installed), prefer the more capable model in doubt, never use a provider that trains on submitted data or keeps it beyond short operational retention, and treat `xhigh` or `max` workers as heavy, one per provider.

## Tools

- If `task` prints a `task:` warning that `model` and `variant` did not apply, use `task_with_model` (plugin `opencode-task-with-model`). Without it, stop and tell the user.
- Sessions need `spawn_session`, `send_agent_message` and `reply` (plugin `marenz/opencode-plugins`). Without them, use tasks or ask the user.

## Setup

- One worker per disjoint scope: never two writers in one checkout.
- Title a spawned session (not a task) `W: <topic>`, or `M: <topic>` for a manager. The global or project `AGENTS.md` may set other prefixes.
- Reusing a warm session, drop its `(DONE) ` and keep `W: `. Ask before reusing a session without `W: `; never reuse an `M: ` one.
- A heavy worker is long-running or on a heavy route. Stay under the provider's limit, counting every worker under you, your workers' workers included.
- Workers may start their own unless their delegation says otherwise: lightweight agents and short tasks on routes not marked heavy freely. For a heavy one, tell your spawner (agent, model, variant, why): a session in a one-line `reply` when starting it, a task in its final answer.
- Tell a worker to slow down when it starts heavy workers too often.

## Cleanup

- Prefix a session's title with `(DONE) ` once its work is verified and finished.
- Remove stale worktrees and branches when the work is merged or abandoned.
