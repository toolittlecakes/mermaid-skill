# mermaid skill

Agent skill for proactive inline [Mermaid](https://mermaid.js.org) visualizations inside ordinary Markdown answers: flowcharts, sequence/class/state/ER/C4 diagrams, mind maps, timelines, Gantt, Kanban, quadrant/pie/XY/radar/Sankey charts, packet diagrams, and more. Picks the diagram grammar by meaning, keeps one idea per diagram, and includes renderer-specific fixes (e.g. the frontmatter needed for the bb IDE).

Works with any agent harness that supports the [Agent Skills](https://agentskills.io) format (Claude Code, Codex, OpenCode, etc.).

## Install

Copy this repo into your agent's skills directory, e.g. for Claude Code:

```bash
git clone https://github.com/toolittlecakes/mermaid-skill.git ~/.claude/skills/mermaid
```

No dependencies — the skill is a single `SKILL.md`.

## Contributing

PRs welcome — this repo is the source of truth for the skill.
