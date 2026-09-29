---
name: agent-messages
description: >
  How OpenCode sessions exchange messages: delegation files, notices, replies
  and final reports. Load when you receive a delegation or an inter-agent
  message, or before messaging another session.
---

# Agent messages

Message directory: `${TMPDIR:-/tmp}/opencode/agent-messages`, unless the global or project `AGENTS.md` sets another.

## Files

- Path: `<dir>/YYYY-MM-DD/YYYYMMDDTHHMMSSZ-NN__from-SRC__to-DST.md`, in UTC, with full session IDs and a counter from `01`. The stem is the message ID.
- A delegation for a session not spawned yet uses `__new-session` instead of `__to-DST`.
- Never overwrite, rename or delete a message; on a collision, increment the counter.
- A reply starts with `In reply to: <parent-stem>`. Find the parent with `ls <dir>/*/<parent-stem>.md`.
- Otherwise only `##` sections: subject, summary and index stay in band.
- Use a file for delegations, final reports, anything over about 15 lines, and logs, diffs or tables. Short acknowledgements and one-line questions stay inline. A worker not allowed to write files reports inline.

## Sending

- To a running session, send through `send_agent_message` or `reply` only a notice: `Subject`, a 1-3 line `Summary`, the absolute `File`, and `Sections` as `L<start>-L<end> <heading>: <when to read it>`. Never paste file contents in band.
- A new session's `spawn_session` prompt is only `Read the full delegation: <absolute path>`.
- A worker session sends its final report through `reply` before its last turn. A task's final report is its final answer, shaped as a notice when long.

```text
Subject: Retry API cannot preserve cancellation
Summary: The proposed retry path retries cancellation, so it is unsafe.
File: /tmp/opencode/agent-messages/2026-09-11/20260911T143112Z-01__from-ses_worker__to-ses_manager.md
Sections:
- L1-L4 Verdict: read to decide whether to discard the option.
- L5-L9 Evidence: read to validate the API behavior.
```

## Receiving

Read the in-band summary first, and only the listed line ranges you need (`Read` with `offset`/`limit`). Use `rg -n '^## ' <file>` only when the index is missing or wrong.
