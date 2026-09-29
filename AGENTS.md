# AGENTS.md

## Testing the skill

- Test spawning from a session: tasks have no `task` tool of their own.
- The recorded `task` input never shows `model` and `variant`, even when they applied; only the `task-model:` line in the task's output (from opencode-task-with-model) is evidence.
- Pick a route unlike what the subagent runs on without them: the agent's own `model`, or, for agents without one such as `general`, the primary agent's configured model and variant, not the variant the calling session currently runs at.
