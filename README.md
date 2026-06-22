# Guardrail

A **local-first AI spend firewall** for coding agents (Claude Code & Codex). It
sits in front of your tools, tracks tokens / cost / sessions, surfaces hidden
costs, and can enforce budget policies — all on your machine. **Metadata only:
no prompts, responses, code, or credentials are ever stored, and nothing leaves
your machine except the calls your tools already make.**

This repository hosts the **prebuilt binaries and installer**. The source is
private; this is the distribution channel for testers.

## Install

macOS & Linux — one command, no GitHub account needed:

```bash
curl -fsSL https://raw.githubusercontent.com/Neatproxy/guardrail-dist/main/install.sh | bash
```

The binary installs to `~/.local/bin/guardrail` (override with
`GUARDRAIL_INSTALL_DIR`; pin a version with `GUARDRAIL_VERSION=vX.Y.Z`). The
download's SHA-256 is verified against `checksums.txt` before install.

> **macOS:** the binary is currently unsigned. The installer clears the
> quarantine flag automatically; if Gatekeeper still complains, run
> `xattr -d com.apple.quarantine ~/.local/bin/guardrail`.

**Windows:** a native build is on the way but **not shipping yet** — use WSL2 (below).

## Windows (via WSL2)

> **A native Windows `guardrail.exe` is in progress — not shipping yet.** Until
> then, run Guardrail under [WSL2](https://learn.microsoft.com/windows/wsl/install),
> where it's just the Linux build and works unchanged.

**Run _everything_ inside the same WSL (Ubuntu) shell — the daemon, the
`guardrail` CLI, _and_ your coding tools (`claude` / `codex`).** A *native
Windows* Claude Code / Codex reads `C:\Users\…` and the Windows network, so it
never sees Guardrail and is **silently not tracked** — even though the dashboard
shows it "connected." Quick check: `which claude` should print a path under
`/home/...`, **not** `/mnt/c/...`.

```powershell
wsl --install        # admin PowerShell, one time; reboot when prompted
```

```bash
# then, inside the Ubuntu (WSL) shell — install + connect + run your tools all here:
curl -fsSL https://raw.githubusercontent.com/Neatproxy/guardrail-dist/main/install.sh | bash
guardrail start
guardrail connect claude-code      # and/or:  guardrail connect codex
# install and run claude / codex INSIDE this same WSL shell, then use them normally
```

Open **http://localhost:4000** in any Windows browser (WSL2 forwards `localhost`
to the Linux VM automatically). If `guardrail` isn't found, add `~/.local/bin` to
your `PATH`; if the dashboard won't open, confirm `wsl -l -v` shows **VERSION 2**
(WSL1 has no localhost forwarding).

## Use

```bash
guardrail start                 # background server + dashboard at http://localhost:4000
guardrail connect claude-code   # keyless passthrough — relays your existing Claude Code login
guardrail connect codex         # keyless usage telemetry (OTEL)
guardrail doctor                # verify the whole setup
```

Open **http://localhost:4000** for live spend, the hidden-cost breakdown,
sessions, and policies. Update any time with **`guardrail update`**. Undo with
`guardrail disconnect <tool>` and `guardrail stop`.

## Privacy

- Data lives in `~/.guardrail/` (local SQLite) — no cloud.
- Bound to `127.0.0.1` by default; reachable only from your machine.
- No prompt / response / credential storage.

## Releases

Binaries for `darwin`/`linux` × `amd64`/`arm64` are attached to each
[release](https://github.com/Neatproxy/guardrail-dist/releases), with a
`checksums.txt`. (Windows: run under WSL2 — see above — until the native build ships.)
