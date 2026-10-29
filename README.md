# lyra-mcp-skills

Plugin for **Lyra (Muse)**, for Claude Code and Codex. One install connects the Lyra MCP server
(`https://mcp-lyra.splitmetrics.com`) and adds skills that tell the agent how to use it: which
calls to make in which order, which models to pick, and known failures. The tool docstrings
in [muse-mcp-server](https://github.com/SplitMetrics/muse-mcp-server) cover single tools. These
skills cover whole workflows.

You need a Lyra account. The first time a Lyra tool runs, your client asks you to sign in to Lyra.
Generation spends your Lyra workspace's credits, and the skills confirm with you before every run.

## Install

### Claude Code

```bash
claude plugin marketplace add SplitMetrics/lyra-mcp-skills
claude plugin install lyra-mcp-skills@lyra
```

Start a new session. To sign in to Lyra right away, run `/mcp` and pick `lyra`.
Update with `claude plugin marketplace update lyra`.

### Codex

```bash
codex plugin marketplace add SplitMetrics/lyra-mcp-skills
codex plugin add lyra-mcp-skills@lyra
codex mcp login lyra
```

Restart Codex. The skills show up as `lyra-mcp-skills:lyra-*`. The plugin also appears under **Plugins** in
the Codex app, where you can enable it instead of running `codex plugin add`.
Update with `codex plugin marketplace upgrade lyra`.

If you added Lyra by hand before (a `[mcp_servers.…]` entry with the same URL in `~/.codex/config.toml`),
remove that entry, so Codex doesn't connect to the same server twice.

### Cursor

`.cursor-plugin/plugin.json` registers the same MCP server for Cursor.

### QA environments

The plugin points at prod. For QA (VPN only), add the server next to it:

```bash
claude mcp add --transport http lyra-qa-muse-1 https://qa-muse-1-mcp-lyra.dev.int.splitmetrics.com
```

```bash
codex mcp add lyra-qa-muse-1 --url https://qa-muse-1-mcp-lyra.dev.int.splitmetrics.com
```

## Skills

Core skills hold the shared rules; the others point to them.

| Skill | Use for |
|---|---|
| `lyra-static-image-generation` | **Core image skill.** Static ads and images end to end: concept, layout, on-image text, realism, models, references, de-slop |
| `lyra-video-ad-generation` | **Core video skill.** Video ads end to end: brief, script, scenes, keyframes, clips, voiceover, music, assembly |
| `lyra-brand-kit` | **Core brand skill.** Build the app's Brand Kit from what the user sends, fill it into Lyra, and the rules every skill follows to read it |
| `lyra-workflow-layout` | Canvas layout for every workflow Claude builds or changes: sections in run order, STEP notes, the team's color code |
| `lyra-video-editor` | Assembly stage of every video: mix, end card, text, captions, pacing, SFX, render check, 59/30/15 s cutdowns |
| `lyra-ugc-video` | UGC-style ads: a realistic person talking to the camera (talking head, testimonial, storytime, street interview) |
| `lyra-video-hooks` | Hook banks from the audience's pains, turned into finished videos for an A/B test |
| `lyra-short-drama` | 60–120 s mini-series story ads, animated or realistic, where the app is the turning point |
| `lyra-ui-motion-video` | App UI in motion, store-style end cards, and App Store / Google Play screenshot sets, with UI text kept exact |
| `lyra-carousel` | 2–10 card carousels that tell one consistent story |
| `lyra-copy-competitor-ad` | Remake a competitor's video ad for the user's app |
| `lyra-copy-competitor-static` | Remake a competitor's static ad or carousel for the user's app |
| `lyra-audience-variations` | The same video ad for another audience: person, wardrobe, setting, market, voice |
| `lyra-ad-concept-multiplier` | New angles on a winning concept, as Lyra variations |
| `lyra-image-resize` | An approved image recomposed for other ratios and placements |
| `lyra-image-translation` | An approved image localized into other languages |
| `lyra-fix` | Fix one part of a finished video or image ad without rebuilding it |
| `lyra-result-check` | Check a finished video or image by looking at it (frames, faces, hands, text, sound), after every final render |
| `lyra-workflow-review` | Explain any workflow in plain words, find what's broken, price a rerun, and fix it on request |

## Layout

```
.claude-plugin/marketplace.json   # marketplace "lyra" (Claude Code and Codex both read it)
.claude-plugin/plugin.json        # Claude Code plugin "lyra-mcp-skills", mcpServers → .mcp.json
.codex-plugin/plugin.json         # Codex plugin manifest: same skills and server, plus its app listing
.mcp.json                         # Lyra MCP server
skills/<name>/SKILL.md            # one folder per skill
```

## Contributing

- The `description` frontmatter decides when the agent loads a skill. Write it as "Use when…" and
  include the phrases users actually say.
- Put in the body what the tool docstrings don't already say: call order, model choice, cost, known
  failures. Do not restate tool schemas.
- Model names change. Every skill points at `list_node_configs` as the source of truth. Update the
  model tables when Lyra's preferred models change.
- On each release, bump `version` in `.claude-plugin/marketplace.json`, `.claude-plugin/plugin.json` and
  `.codex-plugin/plugin.json`.
