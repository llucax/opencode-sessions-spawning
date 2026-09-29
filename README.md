# opencode sessions spawning

Two [OpenCode](https://opencode.ai) skills for delegating work to other sessions and tasks ("workers"):

- `sessions-spawning`, for spawners: which kind of worker to use, how to title and count them, how to pass the model and effort, and how to clean up.
- `agent-messages`, for spawners and workers alike: how sessions exchange delegations, notices, replies and reports through files.

A worker only needs `agent-messages` to read its delegation and report back; a spawner loads both. A single plain `task` call needs neither, only a `model-routing` skill.

## What it expects

- A skill named `model-routing` that picks the model for each worker. The contract is described in the skill: for each worker it gives a `provider/model`, an effort (passed as `variant`), whether the route is heavy with the provider's limit on concurrent heavy workers, and whether it is for bounded work only; or it says no route fits. Write your own, as a table in Markdown or around a script. Without one, `sessions-spawning` falls back to a few generic rules.
- For sessions: the `spawn_session`, `send_agent_message` and `reply` tools from [marenz/opencode-plugins](https://github.com/marenz/opencode-plugins).
- Optionally, [opencode-task-with-model](https://github.com/llucax/opencode-task-with-model), for OpenCode versions where the built-in `task` ignores its `model` and `variant`.
- Optionally, a `quota` tool such as [opencode-quota-agent-tool](https://github.com/llucax/opencode-quota-agent-tool), used only by the fallback rules.

## Configuration

Two local conventions have defaults in the skills, and your global or project `AGENTS.md` can override them:

- The message directory (`agent-messages`), `${TMPDIR:-/tmp}/opencode/agent-messages` by default. Every session that exchanges messages must see it at the same absolute path, so sessions running in separate containers need a shared directory. Set a persistent one if sessions outlive reboots.
- Session title prefixes (`sessions-spawning`): `W: ` for workers, `M: ` for managers and `(DONE) ` for finished sessions. They decide which sessions an agent may reuse, so change them everywhere or not at all.

For example, in `AGENTS.md`:

```markdown
- The inter-agent message directory is `~/shared/agent-messages`.
```

## Installation

Clone the repository and symlink both skills into OpenCode's skills directory:

```sh
git clone https://github.com/llucax/opencode-sessions-spawning ~/opencode-plugins/opencode-sessions-spawning
ln -s ~/opencode-plugins/opencode-sessions-spawning/skills/sessions-spawning ~/.config/opencode/skills/
ln -s ~/opencode-plugins/opencode-sessions-spawning/skills/agent-messages ~/.config/opencode/skills/
```

Update with `git pull` and restart OpenCode.

## License

[MIT](LICENSE)
