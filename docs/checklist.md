# Checklist — Codex port

- [x] Explore upstream plugin structure and Codex plugin system
- [x] Confirm Codex marketplace/plugin schema (`.agents/plugins/marketplace.json`, `source`+`policy`)
- [x] Copy upstream source into repo
- [x] Restructure to Codex-native layout (`plugins/understand-anything/`)
- [x] Write `.agents/plugins/marketplace.json`, `.codex-plugin/plugin.json`, `.claude-plugin/plugin.json`
- [x] Codex-ize skill plugin-root resolution (SKILL_DIR + cache) in understand / -domain / -dashboard
- [x] Switch build to `npx pnpm` (Node-only prerequisite)
- [x] Remove auto-update hook (v1)
- [x] Add README + LICENSE (attribution preserved)
- [x] Verify build + WASM extraction smoke tests
- [x] Push to `PDJ980614/understand-anything-cdx`
- [x] Verify clean install from GitHub
- [ ] (v2) Interactive end-to-end `/understand` run in Codex
- [ ] (v2) Codex-format auto-update hook
