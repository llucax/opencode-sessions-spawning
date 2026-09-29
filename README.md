# opencode sessions spawning

An [OpenCode](https://opencode.ai) skill that tells agents when and how to delegate work to other sessions and tasks ("workers"): which kind of worker to use, how to title and count them, how to pass the model and effort, how to exchange messages through files, and how to clean up.

## What it expects

- A skill named `model-routing` that picks the model for each worker. The contract is described in the skill: for each worker it gives a `provider/model`, an effort (passed as `variant`), whether the route is heavy with the provider's limit on concurrent heavy workers, and whether it is for bounded work only; or it says no route fits. Write your own, as a table in Markdown or around a script. Without one, `sessions-spawning` falls back to a few generic rules.
- For sessions: the `spawn_session`, `send_agent_message` and `reply` tools from [marenz/opencode-plugins](https://github.com/marenz/opencode-plugins).
- Optionally, [opencode-task-with-model](https://github.com/llucax/opencode-task-with-model), for OpenCode versions where the built-in `task` ignores its `model` and `variant`.
- Optionally, a `quota` tool such as [opencode-quota-agent-tool](https://github.com/llucax/opencode-quota-agent-tool), used only by the fallback rules.

## Configuration

Two local conventions have defaults in the skill, and your global or project `AGENTS.md` can override them:

- The message directory, `${TMPDIR:-/tmp}/opencode/agent-messages` by default. Every session that exchanges messages must see it at the same absolute path, so sessions running in separate containers need a shared directory. Set a persistent one if sessions outlive reboots.
- Session title prefixes: `W: ` for workers, `M: ` for managers and `(DONE) ` for finished sessions. They decide which sessions an agent may reuse, so change them everywhere or not at all.

For example, in `AGENTS.md`:

```markdown
- The inter-agent message directory is `~/shared/agent-messages`.
```

## Installation

Clone the repository and symlink the skill into OpenCode's skills directory:

```sh
git clone https://github.com/llucax/opencode-sessions-spawning ~/opencode-plugins/opencode-sessions-spawning
ln -s ~/opencode-plugins/opencode-sessions-spawning/skills/sessions-spawning ~/.config/opencode/skills/
```

Update with `git pull` and restart OpenCode.

## License

[MIT](LICENSE)
