# 🪼 JellyClaw — The Local AI Office

<p align="center">
  <img src="https://raw.githubusercontent.com/JellyClaw-org/.github/main/assets/jellyclaw-logo-pixel.png" alt="JellyClaw" width="560" />
</p>

<p align="center"><b>YOUR OFFICE. YOUR MACHINE. YOUR RULES.</b></p>

<p align="center">
  <a href="https://jellyclaw.in"><img src="https://img.shields.io/badge/WEBSITE-jellyclaw.in-ff4fa3?style=for-the-badge" /></a>
  <a href="https://jellyclaw.in/docs"><img src="https://img.shields.io/badge/DOCS-read-1a1a1a?style=for-the-badge&logo=readthedocs&logoColor=white" /></a>
  <a href="https://discord.gg/GUbgEhHvxt"><img src="https://img.shields.io/badge/DISCORD-join-5865F2?style=for-the-badge&logo=discord&logoColor=white" /></a>
  <a href="https://reddit.com/r/JellyClaw"><img src="https://img.shields.io/badge/REDDIT-r%2FJellyClaw-FF4500?style=for-the-badge&logo=reddit&logoColor=white" /></a>
  <a href="https://x.com/trybild"><img src="https://img.shields.io/badge/X-@trybild-000000?style=for-the-badge&logo=x&logoColor=white" /></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/LICENSE-MIT-blue?style=for-the-badge" /></a>
</p>

**JellyClaw** is a *local AI office* you run on your own machine. You
describe a team in one YAML file — a CEO Agent, Department Heads, and
Worker Agents — and JellyClaw runs that hierarchy on your hardware via
[Ollama](https://ollama.com). Give the CEO a goal; it plans, delegates
down the chain, and reports back to you. Ollama is just the engine —
**the product is the team.**

No cloud. No API bills. No data leaving your machine.

> [!NOTE]
> **Status:** the core engine is in active development. The
> architecture below reflects the current build — some pieces are
> still landing. Follow along in [Issues](../../issues) and
> [Releases](../../releases), or drop into
> [Discord](https://discord.gg/GUbgEhHvxt) /
> [r/JellyClaw](https://reddit.com/r/JellyClaw) — both are just
> getting started, so it's early days and a good time to shape things.

---

## ⚡ Quick Start

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

## 🏢 How it works

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

Each agent runs its own Ollama model — mix and match freely (a fast
small model for the CEO, a coding-tuned model for the worker that
writes code). Results roll back up the chain and land in your
messaging channel.

---

## 📝 Configuration

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
| `channel` | Where you talk to the CEO Agent. `telegram` for now — see [Roadmap](#%EF%B8%8F-roadmap). |
| `ceo` | The top-level agent. Takes your goal, plans, delegates. |
| `departments[].head` | Breaks a department's slice of the goal into worker tasks, reviews results. |
| `departments[].workers[]` | Executes tasks using the `tools` it's allowed to use. |
| `escalation` | Optional fallback to a hosted model (Claude API) for tasks the local model struggles with. Off by default. |

Run `jellyclaw validate` any time to check your config — it points to
the exact field if something's wrong.

---

## 💬 Talking to your team

Once `jellyclaw run` (or the background daemon — see below) is
running, message your CEO Agent directly:

- **Telegram** — message the bot you configured; it replies with
  progress and final results
- **Localhost dashboard** — `jellyclaw dashboard` opens a local,
  read-only view of what every agent is doing right now, plus a chat
  panel to talk to the CEO from your browser (never exposed beyond
  `127.0.0.1`)

Discord support is planned — see [Roadmap](#%EF%B8%8F-roadmap).

---

## 🌙 Running 24/7

The AI team that works while you sleep:

```bash
jellyclaw daemon install   # runs in the background, survives reboots
jellyclaw daemon status
jellyclaw daemon logs
jellyclaw daemon stop
```

Supported on macOS (launchd) and Linux (systemd user service).
Windows daemon support isn't built yet.

Run `jellyclaw doctor` any time to check Ollama connectivity, config
validity, and daemon status in one shot.

---

## 🔧 Tools

Agents can use tools you allow per-worker in the YAML. Built in today:

- **`shell`** — run a shell command (timeout + allowlist, see code
  comments for the security tradeoffs)
- **`file_ops`** — read/write files inside a configured working
  directory only

More tools are straightforward to add — see `jellyclaw/tools/registry.py`.

---

## 🗺️ Roadmap

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

## 🤝 Contributing

Issues and PRs are welcome. Before sending a PR:

1. Open an issue first for anything beyond a small fix, so we can
   align on approach
2. `uv sync && uv run pytest` should pass
3. Keep the channel abstraction intact — the engine should never
   import a specific channel (Telegram, dashboard, etc.) directly,
   only `ChannelAdapter`

See [CONTRIBUTING.md](CONTRIBUTING.md) for the full guide.

<details>
<summary><b>Development setup & verification checklist</b></summary>

```bash
git clone https://github.com/JellyClaw-org/jellyclaw.git
cd jellyclaw
uv sync
uv run pytest -v
```

Manual verification:

- [ ] `uv run jellyclaw init` → pick a template
- [ ] `uv run jellyclaw validate` passes
- [ ] `uv run jellyclaw run` starts without crashing (needs a local Ollama instance)
- [ ] `uv run jellyclaw doctor` reports each check correctly
- [ ] `uv run jellyclaw daemon install` registers a service (macOS/Linux)
- [ ] `uv run jellyclaw dashboard` opens on localhost and shows the agent hierarchy
- [ ] A message sent from the dashboard reaches the CEO the same way a Telegram message would

</details>

---

## 🪼 About

JellyClaw is built by [TryBild](https://github.com/TryBild), a
Mumbai-based product studio. It's the only fully open-source project
we maintain — everything else we build is closed-source commercial
SaaS.

---

<p align="center">
  <b><a href="https://jellyclaw.in">jellyclaw.in</a></b> ·
  <b><a href="https://jellyclaw.in/docs">Docs</a></b> ·
  <b><a href="https://x.com/trybild">@trybild</a></b> ·
  <b><a href="https://discord.gg/GUbgEhHvxt">Discord</a></b> ·
  <b><a href="https://reddit.com/r/JellyClaw">r/JellyClaw</a></b>
  <br /><br />
  MIT Licensed — see <a href="LICENSE">LICENSE</a>
  <br />
  Built in Mumbai, for everywhere. 🪼
</p>
