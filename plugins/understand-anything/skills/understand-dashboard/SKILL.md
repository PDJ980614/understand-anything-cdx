---
name: understand-dashboard
description: Launch the interactive web dashboard to visualize a codebase's knowledge graph
argument-hint: [project-path]
---

# /understand-dashboard

Start the Understand Anything dashboard to visualize the knowledge graph for the current project.

## Instructions

1. Determine the project directory:
   - If `$ARGUMENTS` contains a path, use that as the project directory
   - Otherwise, use the current working directory

2. Check that `.understand-anything/knowledge-graph.json` exists in the project directory. If not, tell the user:
   ```
   No knowledge graph found. Run /understand first to analyze this project.
   ```

3. Find the dashboard code. The dashboard is at `packages/dashboard/` relative to this plugin's root directory. Codex installs the full plugin tree under `~/.codex/plugins/cache/understand-anything-cdx/understand-anything/<version>/`, so derive the root from `SKILL_DIR` first, then fall back to the Codex install cache and other common locations.

   Use the Bash tool to resolve:
   ```bash
   # Codex provides the absolute path of this loaded skill folder when the skill runs.
   # Set SKILL_DIR to that path — the directory that contains this SKILL.md.
   SKILL_DIR="<absolute path to this loaded skill folder>"
   SELF_FROM_SKILLDIR=$([ -n "$SKILL_DIR" ] && cd "$SKILL_DIR/../.." 2>/dev/null && pwd || echo "")

   # Probe the Codex install cache (full plugin tree is materialized there); pick the newest version.
   CODEX_CACHE_ROOT=$(ls -d "$HOME"/.codex/plugins/cache/*/understand-anything/*/ 2>/dev/null | sort -V | tail -1)

   PLUGIN_ROOT=""
   for candidate in \
     "$SELF_FROM_SKILLDIR" \
     "$CODEX_CACHE_ROOT" \
     "${CODEX_PLUGIN_ROOT}" \
     "${CLAUDE_PLUGIN_ROOT}" \
     "$HOME/understand-anything/understand-anything-plugin"; do
     if [ -n "$candidate" ] && [ -d "$candidate/packages/dashboard" ]; then
       PLUGIN_ROOT="$candidate"; break
     fi
   done

   if [ -z "$PLUGIN_ROOT" ]; then
     echo "Error: Cannot find the understand-anything plugin root."
     echo "Checked the SKILL_DIR-derived path, the Codex plugin cache (~/.codex/plugins/cache/...), CODEX_PLUGIN_ROOT, CLAUDE_PLUGIN_ROOT, and ~/understand-anything/understand-anything-plugin."
     echo "Make sure the plugin is installed: codex plugin add understand-anything@understand-anything-cdx"
     exit 1
   fi
   ```

4. Install dependencies and build if needed (uses `npx pnpm`, so only Node.js ≥ 22 is required):
   ```bash
   cd <dashboard-dir> && (npx --yes pnpm@10 install --frozen-lockfile 2>/dev/null || npx --yes pnpm@10 install)
   ```
   Then ensure the core package is built (the dashboard depends on it):
   ```bash
   cd <plugin-root> && npx --yes pnpm@10 --filter @understand-anything/core build
   ```

5. Start the Vite dev server pointing at the project's knowledge graph:
   ```bash
   cd <dashboard-dir> && GRAPH_DIR=<project-dir> npx vite --host 127.0.0.1
   ```
   Run this in the background so the user can continue working.

6. **Capture the access token URL from the server output.** The Vite server prints a line like:
   ```
   🔑  Dashboard URL: http://127.0.0.1:<PORT>?token=<TOKEN>
   ```
   Extract the full URL including the `?token=` parameter. The token is required to access the knowledge graph data — without it the dashboard will show an "Access Token Required" gate.

7. Report to the user, including the full tokenized URL:
   ```
   Dashboard started at http://127.0.0.1:<PORT>?token=<TOKEN>
   Viewing: <project-dir>/.understand-anything/knowledge-graph.json

   The dashboard is running in the background. Press Ctrl+C in the terminal to stop it.
   ```
   **Important:** Always include the `?token=` parameter in the URL you share. If you omit it, the user will be blocked by the token gate and have to manually find the token in the terminal output.

## Notes

- The dashboard auto-opens in the default browser via `--open`
- If port 5173 is already in use, Vite will pick the next available port
- The `GRAPH_DIR` environment variable tells the dashboard where to find the knowledge graph
