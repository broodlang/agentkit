# agentkit

What a Brood program needs to work with a coding agent, whichever model runs it. It knows
nothing of any editor; [bedit-agent](../bedit-agent) is its first user, and zubr can share it.

- **`agentkit/events`**: the ONE vocabulary every backend reports in,
  `[:agent worker kind payload]`, with kinds `:ready` `:text-delta` `:text` `:thinking`
  `:tool-use` `:tool-result` `:turn-done` `:title` `:limits` `:error` `:refused` `:exited`. A front
  end folds these and never learns which backend sent them.
- **`agentkit/tool`**: tools as data (`{:name :description :schema :handler :risk}`), results
  as `{:ok? :output :summary}`. One tool is an MCP catalogue entry (`->mcp-entry`, with a wrap
  for permission and feedback) or an API function definition (`->function-definition`).
- **`agentkit/claude-code`**: Claude Code on the **subscription only**:
  - every `claude` child runs with `ANTHROPIC_API_KEY`, `ANTHROPIC_AUTH_TOKEN`,
    `ANTHROPIC_BASE_URL` and the Bedrock / Vertex / Foundry switches removed
    (`subscription-env`);
  - `claude auth status` must say `claude.ai` before a session starts
    (`subscription-problem`);
  - the session's own `init` event must say `apiKeySource: none` (`init-problem`), which
    catches a settings file's `apiKeyHelper`. A session failing any check is `:refused`.

  `start` runs a headless stream-json session in a worker (`[:prompt text]`, `[:interrupt]`,
  `[:stop]`). The text deltas of one read are joined into one event. With `:owner`, the session
  ends when that process does. `write-mcp-config` puts the MCP servers' config (and their
  tokens) in a file only you can read, for `:mcp-config-file`. `interactive-command` is the
  shell line for the CLI's own interface under the same rules. `history` reads a past session
  back from the CLI's log.
- **`agentkit/claude-log`**: every Claude Code session on the machine, from the JSON-lines log
  the CLI writes as it works (`<config-dir>/projects/<cwd>/<id>.jsonl`), whoever started it.
  `config-dirs` finds every account (`~/.claude*`, `CLAUDE_CONFIG_DIR`); `sessions` lists the
  logs that changed lately with what each is about and whether it is mid-turn, read from the
  tail alone; `read-new` reads what a log gained since a cursor as the shared vocabulary
  (brood's `file/slurp-bytes-from`, a half-written line held back); `follow` is a process that
  sends it as `[:agent-log follower {:events :reset?}]` as soon as the OS reports the log changed
  (brood's `file/watch`), with a slow look as a backstop. A subagent's lines are left out. Needs
  brood 0.58 (`file/slurp-bytes-from`, ADR-781; `file/watch`, ADR-782).

```bash
nest test   # 29 tests against a stand-in CLI (tests/fixtures/fake-claude); no turn is spent
```
