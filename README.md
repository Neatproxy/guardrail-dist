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
curl -fsSL https://raw.githubusercontent.com/shreyas2231/guardrail-dist/main/install.sh | bash
```

The binary installs to `~/.local/bin/guardrail` (override with
`GUARDRAIL_INSTALL_DIR`; pin a version with `GUARDRAIL_VERSION=vX.Y.Z`). The
download's SHA-256 is verified against `checksums.txt` before install.

> **macOS:** the binary is currently unsigned. The installer clears the
> quarantine flag automatically; if Gatekeeper still complains, run
> `xattr -d com.apple.quarantine ~/.local/bin/guardrail`.

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
[release](https://github.com/shreyas2231/guardrail-dist/releases), with a
`checksums.txt`.
