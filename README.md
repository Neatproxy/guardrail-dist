# Guardrail

A **local-first AI spend firewall** for coding agents (Claude Code & Codex). It
sits in front of your tools, tracks tokens / cost / sessions, surfaces hidden
costs, and can enforce budget policies — all on your machine. **Metadata only:
no prompts, responses, code, or credentials are ever stored, and nothing leaves
your machine except the calls your tools already make.**

This repository hosts the **prebuilt binaries and installer**. The source is
private; this is the distribution channel for testers. Guardrail asks to sit
in front of your coding agent, so before you run anything here, please read
**[Trust and security](#trust-and-security)** below. It states plainly who
publishes this, what the binary can see, what it keeps, and what is not yet
in place.

## Install

macOS & Linux — one command, no GitHub account needed:

```bash
curl -fsSL https://raw.githubusercontent.com/Neatproxy/guardrail-dist/main/install.sh | bash
```

The binary installs to `~/.local/bin/guardrail` (override with
`GUARDRAIL_INSTALL_DIR`; pin a version with `GUARDRAIL_VERSION=vX.Y.Z`). The
download's SHA-256 is verified against `checksums.txt` before install.

> **macOS:** this build is **not yet signed or notarized** by Apple, and the
> installer deliberately does **not** clear the quarantine flag for you. If
> Gatekeeper blocks the binary, allowing it is your decision to make:
> `xattr -d com.apple.quarantine ~/.local/bin/guardrail`.
> Signing and notarization are on the roadmap.

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

## Trust and security

Guardrail relays traffic between your coding agent and the model provider. That
is a position of real trust, so here is the honest version, including the parts
that count against us.

**Who publishes this.** Guardrail is built by NeatProxy, an independent project
at an early stage: no outside investors, no registered company yet. It is the
same people behind [neatproxy.com](https://neatproxy.com). Security questions,
disclosures, and data requests go to **admin@neatproxy.com** and are answered by
a person.

**What the binary can see, and what it keeps.** It necessarily sees requests
and responses in memory while relaying them, and with keyless passthrough it
relays your existing tool login. It parses a handful of metadata fields and
discards the bytes. Not stored anywhere, on disk or off it: prompts, responses,
source code, tool inputs and outputs, shell command text, or API keys (yours or
upstream). Kept: model, provider, endpoint, token counts, cost estimates,
latency, status, and session/project ids. The full field-by-field table is in
the [privacy model](https://neatproxy.com/docs/privacy).

That claim is enforced by the build, not just asserted in a document: the test
suite sends a known secret prompt through both the proxy and the hook receiver,
then scans every byte SQLite wrote to disk (database, WAL, and shared memory)
for that secret. If it ever appears, the build fails.

**Where data lives.** `~/.guardrail/` (local SQLite), bound to `127.0.0.1`, no
cloud. Cloud sync is opt-in and off by default on Free, Trial, and Pro; when it
is on, it carries the same metadata described above and never prompts,
responses, code, or your local salt.

**What is not yet in place.** We would rather you hear this from us than find
it yourself:

- The **source is closed**, so the guarantees above cannot be independently
  audited today. Verify what you can: the installer is right here, plain text.
- The macOS binaries are **not signed or notarized**. The installer no longer
  clears the quarantine flag on your behalf.
- `checksums.txt` ships in the **same release** as the binary. It protects you
  against a corrupted or truncated download, not against a bad publisher. Only
  signing addresses the second one, and it is on the roadmap.

**Read further.** [Security overview](https://neatproxy.com/docs/security)
covers the threat model, an external review, and the tradeoffs we accept.
[Privacy model](https://neatproxy.com/docs/privacy) covers what is recorded and
what never is. [Machine security](https://neatproxy.com/docs/machine-security)
covers how a machine is identified and revoked.

**If you would rather not take this on:** you do not have to. `/usage` inside
Claude Code and the Anthropic console usage page cover basic spend visibility
with no third-party binary at all. We would rather you use those than install
something you are not comfortable with.

## Releases

Binaries for `darwin`/`linux` × `amd64`/`arm64` are attached to each
[release](https://github.com/Neatproxy/guardrail-dist/releases), with a
`checksums.txt`. (Windows: run under WSL2 — see above — until the native build ships.)
