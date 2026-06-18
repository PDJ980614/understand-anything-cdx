# Context Notes — Codex port of Understand-Anything

Decisions made while porting `understand-anything` (Claude Code plugin, by Egonex-AI, MIT)
to a Codex CLI plugin. Newest context at the bottom.

## Goal

A Codex-CLI-installable plugin that mirrors Understand-Anything. The model-agnostic core
(graph engine + WASM tree-sitter + React dashboard) is reused as-is; only the
harness-specific layer was adapted. (User decisions: Codex-only standalone repo, reuse
existing core, exclude auto-update hook for v1, repo `PDJ980614/understand-anything-cdx`,
build locally under `~/understand-anything-codex`.)

## Key findings about Codex's plugin system (CLI 0.140.0)

- Native plugin system: `codex plugin marketplace add <owner/repo|url|local>` then
  `codex plugin add <plugin>@<marketplace>`.
- **Marketplace manifest must be `<repo-root>/.agents/plugins/marketplace.json`** — NOT
  `.claude-plugin/marketplace.json`. The Claude-format marketplace was silently read as
  "0 plugins". This was the single biggest blocker.
- Each marketplace plugin entry needs an **object** `source` and a **`policy`** block:
  ```json
  { "name": "understand-anything",
    "source": { "source": "local", "path": "./plugins/understand-anything" },
    "policy": { "installation": "AVAILABLE", "authentication": "ON_INSTALL" },
    "category": "Developer Tools" }
  ```
- Plugins live under `plugins/<name>/`, each with `.codex-plugin/plugin.json`
  (fields: `skills`, `mcpServers`, `apps`, `hooks`, `interface`). A `.claude-plugin/plugin.json`
  is kept inside the plugin dir for cross-harness metadata but is not what Codex reads to install.
- Plugin name must differ from / be addressed as `<plugin>@<marketplace>`.
- On install Codex materializes the **full plugin tree** at
  `~/.codex/plugins/cache/<marketplace>/<plugin>/<version>/` — so `packages/` is reachable
  from a skill at `<root>/skills/<x>/` via `../..`.
- Codex skills reference bundled files relative to `SKILL_DIR` (the loaded skill folder,
  whose absolute path Codex provides at load time). Convention seen in other plugins:
  `SKILL_DIR="<absolute path to this loaded skill folder>"`.
- Authoritative schema source: the bundled `plugin-creator` skill
  (`~/.codex/.../skills/plugin-creator/references/plugin-json-spec.md`).

## What changed vs upstream

- Repo layout → Codex-native (`.agents/plugins/marketplace.json` + `plugins/understand-anything/`).
- `skills/{understand,understand-domain,understand-dashboard}/SKILL.md`: plugin-root
  resolution rewritten to derive from `SKILL_DIR` + Codex install cache; removed
  `~/.agents`/`~/.copilot` symlink probing.
- Build step `pnpm ...` → `npx --yes pnpm@10 ...`, so **Node.js ≥ 22 is the only prerequisite**
  (pnpm auto-fetched). Native tree-sitter build scripts are skipped by pnpm v10; runtime uses
  WASM grammars, so no native toolchain is needed.
- Removed `hooks/` (Claude-format auto-update hook) for v1.
- Added README + LICENSE (dual attribution: Egonex-AI original + PDJ980614 port).

## Verification done

- `npx pnpm@10 install` (28s, no native build) + `tsc` core build → OK.
- `@understand-anything/core` resolves and imports.
- `scan-project.mjs` on a tiny project → 3 files scanned.
- `extract-structure.mjs` (WASM tree-sitter) → extracted `add`/`Calc` (Python), `greet` (JS).
- Fresh install from GitHub: skills updated, hooks gone, `packages/core` present.

## Not done / future (v2)

- End-to-end interactive `/understand` run inside a Codex session (needs API + interactivity).
- Port the auto-update hook to Codex's `hooks/hooks-codex.json` format.
- Optional: vendor a prebuilt core to skip even the first-run build.
- The `.mjs` scripts still contain "Claude Code / Copilot" comments (harmless; they
  self-resolve `pluginRoot = __dirname/../..`).
