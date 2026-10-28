# Opemhnd-rea-guffy-

OpenHands workspace with [REA](https://github.com/morluto/rea) (reverse engineer
anything) installed, built, and connected to the OpenHands agent as both an MCP
server and a skill.

## What is REA?

REA is a reverse-engineering toolkit that ships as a CLI and an MCP server. It
analyzes shipped artifacts — native binaries, Electron/JavaScript applications,
managed (.NET) assemblies, packaged installers, firmware, web bundles, and
Android packages — and produces Evidence-backed results (decompilation,
inventory, version comparison, runtime capture).

- Upstream: https://github.com/morluto/rea
- Docs: https://rea.tools/

## What is installed here

| Component | Location | Notes |
| --- | --- | --- |
| REA source | `/workspace/project/rea` | v6.8.0, cloned from `morluto/rea` |
| `rea` CLI | `/usr/local/bin/rea` | Global npm install, reports `6.8.0` |
| Node.js | `/opt/node`, linked into `/usr/local/bin` | v24.22.1 (npm 11.19.1) |
| Ghidra | `/opt/ghidra/ghidra_12.1.4_PUBLIC` | Native binary decompiler provider |
| JDK | `/usr/lib/jvm/java-21-openjdk-amd64` | OpenJDK 21, required by Ghidra 12.1.4 |
| MCP registration | `~/.openhands/settings.json` | stdio server named `rea`, 231 tools |
| Skill (user) | `~/.agents/skills/reverse-engineer-anything` | Loaded by the agent server |
| Skill (project) | `.agents/skills/reverse-engineer-anything` | Travels with this repo |

## Integration details

### MCP server

REA is registered with the OpenHands agent server as a stdio MCP server:

```json
{
  "rea": {
    "transport": "stdio",
    "command": "/usr/local/bin/node",
    "args": [
      "/opt/node/lib/node_modules/rea-agents/scripts/rea.mjs",
      "mcp"
    ],
    "env": {
      "GHIDRA_INSTALL_DIR": "/opt/ghidra/ghidra_12.1.4_PUBLIC",
      "JAVA_HOME": "/usr/lib/jvm/java-21-openjdk-amd64"
    },
    "description": "REA - reverse engineering tools",
    "enabled": true
  }
}
```

The server advertises 231 tools and 6 guided prompts.

### Skill

The bundled `reverse-engineer-anything` skill is installed at both user and
project scope. It teaches the agent when to reach for REA and how to read
Evidence, without running diagnostics before every investigation.

### Environment

`/etc/profile.d/rea.sh` (sourced from `~/.bashrc`) exports `PATH`,
`GHIDRA_INSTALL_DIR`, and `JAVA_HOME` for shell use.

## Verification

```bash
rea --version          # 6.8.0
rea mcp doctor         # healthy: 231/231 tools, 6/6 prompts
rea doctor             # healthy, environment_healthy
```

Functional checks performed:

- JavaScript/Electron analysis: correctly resolved a `contextBridge` preload
  bridge to its `ipcRenderer.invoke` channel.
- Native analysis: decompiled an ELF `verify_license` function via Ghidra,
  recovering the DJB2 checksum comparison against `0x1a2b3c4d`.

## Optional providers

These are not installed because they require caller-supplied licensed or
third-party engines:

| Provider | Enables | Requires |
| --- | --- | --- |
| IDA | IDA Pro decompilation | `REA_IDA_MCP_CONFIG` |
| jadx | Android APK decompilation | `REA_JADX_MCP_JAR` (jadx-headless-mcp 0.7.1) |
| kuna | Kernel/binary analysis | `REA_KUNA_COMMAND` |
| adb | Device capture | Android platform tools |

Set the matching environment variable and restart REA to enable one.
