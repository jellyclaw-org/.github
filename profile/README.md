<div align="center">

<img src="assets/jellyclaw-logo-pixel.png" alt="JellyClaw" width="560" />

**The AI team that works while you sleep.**

A hierarchy of local AI agents — CEO → Department Heads → Workers —
running entirely on your own hardware. Self-hosted. YAML-configured.
Ollama-powered. Open source.

[![Website](https://img.shields.io/badge/jellyclaw.in-ff4fa3?style=for-the-badge&logo=googlechrome&logoColor=white)](https://jellyclaw.in)
[![Docs](https://img.shields.io/badge/Docs-1a1a1a?style=for-the-badge&logo=readthedocs&logoColor=white)](https://jellyclaw.in/docs)
[![X](https://img.shields.io/badge/@trybild-000000?style=for-the-badge&logo=x&logoColor=white)](https://x.com/trybild)
[![Discord](https://img.shields.io/badge/Discord-5865F2?style=for-the-badge&logo=discord&logoColor=white)](https://discord.gg/GUbgEhHvxt)
[![Reddit](https://img.shields.io/badge/r%2FJellyClaw-FF4500?style=for-the-badge&logo=reddit&logoColor=white)](https://reddit.com/r/JellyClaw)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](LICENSE)

</div>

---

## What is JellyClaw?

JellyClaw is a local multi-agent orchestration tool. You describe a
team in a YAML file — a CEO Agent, one or more Department Heads, and
Worker Agents under each — and JellyClaw runs that hierarchy on your
own machine using [Ollama](https://ollama.com). The CEO takes a goal,
delegates it down the chain, and reports results back to you.

No cloud, no data leaving your machine, no subscription. Just your
hardware and your models.

> **Status:** the core engine is in active development. The
> architecture below reflects the current build — some pieces are
> still landing. Follow along in [Issues](../../issues) and
> [Releases](../../releases), or drop into [Discord](https://discord.gg/GUbgEhHvxt)
> / [r/JellyClaw](https://reddit.com/r/JellyClaw) — both are just
> getting started, so it's early days and a good time to shape things.

---

## How it works

```
        You
         │
         ▼
   ┌───────────┐
   │ CEO Agent │  ← takes your goal, makes a plan, delegates
   └─────┬─────┘
         │
   ┌─────┴─────┬─────────────┐
   ▼           ▼             ▼
┌────────┐ ┌────────┐   ┌────────┐
│  Dept  │ │  Dept  │   │  Dept  │  ← routes tasks, reviews work
│  Head  │ │  Head  │   │  Head  │
└───┬────┘ └───┬────┘   └───┬────┘
    │          │            │
 ┌──┴──┐    ┌──┴──┐      ┌──┴──┐
 ▼     ▼    ▼     ▼      ▼     ▼
Worker Worker Worker Worker Worker Worker   ← execute with tools
```

Each agent runs its own Ollama model — mix and match freely (e.g. a
fast small model for the CEO, a coding-tuned model for a worker that
writes code). Results roll back up the chain and land in your
messaging channel.

---

## Quick Start

**Requirements:** Python 3.11+, [uv](https://docs.astral.sh/uv/), and
[Ollama](https://ollama.com) running locally.

```bash
# Run without installing anything
uvx jellyclaw init

# Pick a template (dev-team or sales-office), then:
uvx jellyclaw validate
uvx jellyclaw run
```

Prefer pip?

```bash
pip install jellyclaw
jellyclaw init
```

Building from source (contributors):

```bash
git clone https://github.com/JellyClaw-org/jellyclaw.git
cd jellyclaw
uv sync
uv run jellyclaw init
```

---

## Configuration

Everything is one YAML file. Here's the `dev-team` template:

```yaml
channel: telegram

ceo:
  model: llama3.1:8b
  name: "The Boss"

departments:
  - name: engineering
    head:
      model: llama3.1:8b
    workers:
      - name: coder
        model: qwen2.5-coder:7b
        tools: [shell, file_ops]
      - name: reviewer
        model: llama3.1:8b
        tools: [file_ops]

escalation:
  enabled: false        # optional: hand off hard tasks to Claude API
  provider: anthropic
  model: claude-sonnet-4-6
```

| Field | What it does |
|---|---|
| `channel` | Where you talk to the CEO Agent. `telegram` for now — see [Roadmap](#roadmap). |
| `ceo` | The top-level agent. Takes your goal, plans, delegates. |
| `departments[].head` | Breaks a department's slice of the goal into worker tasks, reviews results. |
| `departments[].workers[]` | Executes tasks using the `tools` it's allowed to use. |
| `escalation` | Optional fallback to a hosted model (Claude API) for tasks the local model struggles with. Off by default. |

Run `jellyclaw validate` any time to check your config — it points to
the exact field if something's wrong.

---

## Talking to your team

Once `jellyclaw run` (or the background daemon — see below) is
running, message your CEO Agent directly:

- **Telegram** — message the bot you configured; it replies with
  progress and final results
- **Localhost dashboard** — `jellyclaw dashboard` opens a local,
  read-only view of what every agent is doing right now, plus a chat
  panel to talk to the CEO from your browser (never exposed beyond
  `127.0.0.1`)

Discord support is planned — see [Roadmap](#roadmap).

---

## Running 24/7

```bash
jellyclaw daemon install   # runs jellyclaw in the background, survives reboots
jellyclaw daemon status
jellyclaw daemon logs
jellyclaw daemon stop
```

Supported on macOS (launchd) and Linux (systemd user service).
Windows daemon support isn't built yet.

Run `jellyclaw doctor` any time to check Ollama connectivity, config
validity, and daemon status in one shot.

---

## Tools

Agents can use tools you allow per-worker in the YAML. Built in today:

- **`shell`** — run a shell command (timeout + allowlist, see code
  comments for the security tradeoffs)
- **`file_ops`** — read/write files inside a configured working
  directory only

More tools are straightforward to add — see `jellyclaw/tools/registry.py`.

---

## Roadmap

JellyClaw ships in phases, in the open:

- [x] Core engine — CEO → Department Head → Worker hierarchy
- [x] Telegram channel
- [x] 24/7 background daemon
- [x] Localhost dashboard — live agent status + chat with the CEO
- [ ] Discord channel
- [ ] Additional built-in tools
- [ ] Parallel worker execution

Nothing here is promised on a date — this is a small team building in
public. [Watch the repo](../../subscription) or check
[Releases](../../releases) for real progress.

---

## Contributing

Issues and PRs are welcome. Before sending a PR:

1. Open an issue first for anything beyond a small fix, so we can
   align on approach
2. `uv sync && uv run pytest` should pass
3. Keep the channel abstraction intact — the engine should never
   import a specific channel (Telegram, dashboard, etc.) directly,
   only `ChannelAdapter`

See [CONTRIBUTING.md](CONTRIBUTING.md) for the full guide.

---

## Development

```bash
git clone https://github.com/JellyClaw-org/jellyclaw.git
cd jellyclaw
uv sync
uv run pytest -v
```

Manual verification checklist:

- [ ] `uv run jellyclaw init` → pick a template
- [ ] `uv run jellyclaw validate` passes
- [ ] `uv run jellyclaw run` starts without crashing (needs a local Ollama instance)
- [ ] `uv run jellyclaw doctor` reports each check correctly
- [ ] `uv run jellyclaw daemon install` registers a service (macOS/Linux)
- [ ] `uv run jellyclaw dashboard` opens on localhost and shows the agent hierarchy
- [ ] A message sent from the dashboard reaches the CEO the same way a Telegram message would

---

## About

JellyClaw is built by [TryBild](https://github.com/TryBild), a
Mumbai-based product studio. It's the only fully open-source project
we maintain — everything else we build is closed-source commercial
SaaS.

---

<div align="center">

**[jellyclaw.in](https://jellyclaw.in)** · **[Docs](https://jellyclaw.in/docs)** · **[@trybild](https://x.com/trybild)** · **[Discord](https://discord.gg/GUbgEhHvxt)** · **[r/JellyClaw](https://reddit.com/r/JellyClaw)**

MIT Licensed — see [LICENSE](LICENSE)

</div>
