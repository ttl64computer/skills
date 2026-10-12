# ttl64 skills

Agent Skill and MCP connection for [ttl64](https://ttl64.com): short-lived Kubernetes clusters that an AI agent can create, use and let expire.

## Claude Code

```sh
/plugin marketplace add ttl64computer/skills
/plugin install ttl64@ttl64
```

The plugin brings the `ttl64` skill and the `https://mcp.ttl64.com/mcp` server (sign in with OAuth on first use).

## claude.ai and Claude Desktop

Settings → Connectors → Add custom connector → `https://mcp.ttl64.com/mcp`. Upload `plugins/ttl64/skills/ttl64` as a skill if you want the guidance too.

## Codex

Copy `plugins/ttl64/skills/ttl64` into your Codex skills folder, and add the server to `~/.codex/config.toml` (see https://app.ttl64.com/keys/ai for the exact snippet; create a key at https://app.ttl64.com/keys).

## Grok

Grok reads MCP servers only: pass `https://mcp.ttl64.com/mcp` as a remote MCP tool with an API key in `authorization`. The server sends the same guidance as the skill.
