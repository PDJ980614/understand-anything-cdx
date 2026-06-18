# Understand Anything — Codex CLI port

AI-powered codebase understanding for the **Codex CLI**. Analyze any project into an
interactive knowledge graph, then explore architecture, domains, diffs, and onboarding
guides — all from inside Codex.

This is a Codex-harness port of [Egonex-AI/Understand-Anything](https://github.com/Egonex-AI/Understand-Anything)
(MIT). The model-agnostic core (knowledge-graph engine, tree-sitter analysis via WASM,
and the React dashboard) is reused unchanged; only the plugin manifest, marketplace
layout, and the skills' plugin-root/build resolution were adapted for Codex.

## Requirements

- **Codex CLI** ≥ 0.140
- **Node.js ≥ 22** — used to run the analysis scripts. `pnpm` is fetched automatically
  via `npx`, so you do **not** need to install it yourself.

## Install

```bash
codex plugin marketplace add PDJ980614/understand-anything-cdx
codex plugin add understand-anything@understand-anything-cdx
```

On the first `/understand` run the plugin builds its core package once
(`npx pnpm install` + build) inside the install cache. This needs network access the
first time.

## Usage

| Skill | What it does |
|-------|--------------|
| `/understand` | Analyze the codebase into `.understand-anything/knowledge-graph.json` |
| `/understand-dashboard` | Launch the interactive dashboard for the graph |
| `/understand-explain` | Deep-dive a specific file, function, or module |
| `/understand-diff` | Analyze a git diff / PR: what changed, impact, risks |
| `/understand-domain` | Extract business-domain knowledge and a domain flow graph |
| `/understand-onboard` | Generate an onboarding guide for new team members |
| `/understand-chat` | Ask questions about the codebase using the graph |
| `/understand-knowledge` | Build a knowledge graph from a markdown knowledge base |

Example:

```
/understand
/understand-dashboard
```

## Notes on this port

- **Codex-native layout.** The plugin lives at `plugins/understand-anything/` and is
  published through `.agents/plugins/marketplace.json`.
- **Plugin-root resolution** in the skills derives the root from the loaded skill folder
  (`SKILL_DIR`) and the Codex install cache (`~/.codex/plugins/cache/...`).
- **Build step** uses `npx pnpm`, so Node.js is the only prerequisite.
- **Auto-update hook excluded (v1).** The Claude-style commit hook that rebuilt the graph
  automatically is not bundled. `/understand --auto-update` still records the preference,
  and re-running `/understand` updates the graph incrementally.

## Credits

- Original project: **Understand-Anything** by Egonex-AI — https://github.com/Egonex-AI/Understand-Anything
- Codex CLI port: PDJ980614

Licensed under the MIT License. See [LICENSE](./LICENSE).
