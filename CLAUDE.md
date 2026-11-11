# CLAUDE.md: working on agentkit

agentkit is the agent plumbing a Brood program shares, whatever drives it (bedit-agent today,
zubr next). It must never know about an editor.

- `agentkit/events`: the ONE vocabulary every backend reports in. A new kind is a decision:
  add it to `kinds` and to the module doc, and make every front end fold it (or ignore it).
- `agentkit/tool`: tools as data, and results `{:ok? :output :summary}`. One tool serves MCP
  (`->mcp-entry`) and API function calling (`->function-definition`).
- `agentkit/claude-code`: the Claude Code backend, **subscription only**. Three checks:
  - API-key variables are removed from every child (`subscription-env`);
  - `claude auth status` must say `claude.ai`, run in the session's own directory;
  - the session's `init` event must say `apiKeySource: none`.

  Never weaken one, and never put a token on a command line (`write-mcp-config`).
- **Tests use a fake CLI** (`tests/fixtures/fake-claude`), which behaves as the real one: no
  init until the first message. A test never starts the real `claude`. Pass `:program`, since
  a dynvar binding does not reach a spawned worker.
- Brood conventions: `../brood/docs/brood-for-claude.md`. Typed parameters rather than `sig`;
  no abbreviated names; docstring examples run.
