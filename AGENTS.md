# AGENTS.md

Repository-specific knowledge for the OpenHands workspace `Opemhnd-rea-guffy-`.

## Purpose

This workspace hosts a built and integrated copy of REA (reverse engineer
anything). See `README.md` for the full inventory.

## Layout

- `/workspace/project/rea` — REA v6.8.0 source checkout (TypeScript monorepo).
  Not committed here; it is a separate clone of `morluto/rea`.
- `.agents/skills/reverse-engineer-anything` — REA skill bundled with this repo.

## Toolchain

- Node.js v24.22.1 at `/opt/node`, linked into `/usr/local/bin`.
- `rea` CLI at `/usr/local/bin/rea` (global npm install of the local checkout).
- Ghidra 12.1.4 at `/opt/ghidra/ghidra_12.1.4_PUBLIC`; JDK 21 at
  `/usr/lib/jvm/java-21-openjdk-amd64`. Both are required for native binary
  analysis and are exported by `/etc/profile.d/rea.sh`.

## Working with REA

- Prefer the `rea` CLI for shell work and the `rea` MCP server for agent work.
- Do not run `rea doctor` or `rea setup` before every task; they are for
  diagnosis and repair. Use `rea mcp doctor` to validate the MCP adapter.
- REA writes Evidence, not guesses. When a claim depends on a shipped binary or
  package, cite the Evidence record rather than paraphrasing it.

## Rebuilding REA

```bash
cd /workspace/project/rea
npm ci
npm run build:cached
sudo env "PATH=/usr/local/bin:$PATH" npm install -g . --no-audit --no-fund
```

`npm install -g .` does not run the package's `prepare` script by default; the
already-built `dist/` is used. Rebuild before reinstalling if source changed.

## Gotchas

- Ghidra archives extracted without `unzip` lose the executable bit on
  `support/analyzeHeadless` and the native `decompile` binary. Restore modes
  from the zip's `external_attr` field if `rea doctor` reports
  `executable_missing`.
- OpenHands is not a client REA's `rea setup --client` knows about. The MCP
  registration here was written directly through the agent-server settings API.
- The REA plugin route (`github:morluto/rea`) loads but contributes nothing to
  OpenHands: its MCP declaration is Claude Code–specific and its `skills`
  manifest points at `./skills`, which does not exist at the repo root. Use the
  MCP registration plus the `.agents/skills` copy instead.
