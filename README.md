# lyra-mcp-skills

Claude Code plugin marketplace for **Lyra (Muse)**. One install connects the Lyra MCP server
(`https://mcp-lyra.splitmetrics.com`) and adds skills that tell the agent how to use it: which
calls to make in which order, which models to pick, and known failures. The tool docstrings
in [muse-mcp-server](https://github.com/SplitMetrics/muse-mcp-server) cover single tools. These
skills cover whole workflows.

## Install

```bash
claude plugin marketplace add SplitMetrics/lyra-mcp-skills
claude plugin install lyra-mcp-skills@lyra
```

Start a new session after installing. Update with `claude plugin marketplace update lyra`.

The repo is private, so `marketplace add` uses your own GitHub credentials. It works for
SplitMetrics employees with access to the repo.

### QA environments

The plugin points at prod. For QA (VPN only), add the server next to it:

```bash
claude mcp add --transport http lyra-qa-muse-1 https://qa-muse-1-mcp-lyra.dev.int.splitmetrics.com
```

### Cursor / Codex

`.cursor-plugin/plugin.json` and `.codex/config.toml` register the same MCP server for those clients.

## Skills

| Skill | Use for |
|---|---|
| `lyra-start-here` | Routing plus the rules every workflow follows (credits, run order, polling) |
| `lyra-templates` | Lyra's producer-built templates (Hook, Talking Head, UGC…), copied and adapted to the user's product |
| `lyra-generate` | Images or videos from a prompt, single or batch |
| `lyra-image-to-video` | Animating an image as first frame, last frame or reference |
| `lyra-talking-head` | Face + voiceover → lip-sync video, and standalone voiceover or music |
| `lyra-adapt-image` | Resizing to other aspect ratios and translating on-image text |
| `lyra-video-ad` | Multi-scene ad: script → keyframes → clips → voiceover/music → composer |

## Layout

```
.claude-plugin/marketplace.json   # marketplace "lyra"
.claude-plugin/plugin.json        # plugin "lyra-mcp-skills", mcpServers → .mcp.json
.mcp.json                         # Lyra MCP server
skills/<name>/SKILL.md            # one folder per workflow; long docs in reference/*.md
```

## Contributing

**Producer notes for templates** go in `skills/lyra-templates/reference/<template-slug>.md`, in the format
described in [that folder's README](skills/lyra-templates/reference/README.md). This is where the creative
producer's knowledge lives: why a template works, what is safe to change, and known failures.

- The `description` frontmatter decides when the agent loads a skill. Write it as "Use when…" and
  include the phrases users actually say.
- Put in the body what the tool docstrings don't already say: call order, model choice, cost, known
  failures. Do not restate tool schemas.
- Model names change. Every skill points at `list_node_configs` as the source of truth. Update the
  model tables when Lyra's preferred models change.
- On each release, bump `version` in both `.claude-plugin/marketplace.json` and `.claude-plugin/plugin.json`.
