# This Environment

A factual snapshot of the machine and Hermes profile documented here. Every value below
was read from the live system, not assumed.

---

## Host

| Fact | Value |
|---|---|
| Environment | WSL2 (Windows Subsystem for Linux) |
| Distribution | Kali GNU/Linux Rolling (2026.3) |
| Kernel | `6.18.33.2-microsoft-standard-WSL2` |
| Architecture | x86_64 |
| Hostname | `Mani-PC` |
| User | `mani` |
| Home | `/home/mani` |
| Python | 3.14.7 |

> Kali is a Debian-family distribution. Its package repositories do **not** include the
> GitHub CLI (`gh`), which shaped how it was installed here — see `OPERATIONS.md`.

## Hermes profile

| Fact | Value |
|---|---|
| `HERMES_HOME` | `/home/mani/.hermes` |
| Active profile | `default` |
| Model | `deepseek/deepseek-v4.1-flash` |
| Provider | Nous Portal (`nous`) |
| Platform (this session) | `cli` |

Other profiles, if any, would live under `/home/mani/.hermes/profiles/<name>/`, each with
its own skills, plugins, cron jobs, and memories — deliberately isolated from one another.

## What lives under `~/.hermes`

Selected entries, to orient a reader:

```
~/.hermes/
├── config.yaml          # user settings
├── .env                 # secrets only
├── SOUL.md              # persona / standing instruction file
├── skills/              # procedural memory (SKILL.md files)
├── memories/            # durable facts
├── sessions/            # conversation history
├── state.db             # session/state database
├── cron/                # scheduled jobs
├── plugins/             # installed extensions
├── logs/                # `hermes logs`
├── kanban/ kanban.db    # kanban board data
├── gateway.*            # gateway process state / socket
├── cache/ scratch/      # transient working files (pruned when idle)
└── source/ installs/    # source tree / install bookkeeping
```

## Project source tree

The Hermes source is checked out at `/home/mani/hermes-agent`, tracking
`https://github.com/NousResearch/hermes-agent.git` (the public upstream, not a personal
fork). At the time of writing the checkout sits at commit `05eecbcd97`
("chore(memory): remove the bundled openviking provider", 2026-10-06).

## GitHub

| Fact | Value |
|---|---|
| Account | [maninejad](https://github.com/maninejad) (Mani Nejad) |
| Auth method | `gh` CLI, device-flow OAuth |
| Credential store | `/home/mani/.config/gh/hosts.yml` (mode 600) |
| Token scopes | `repo`, `read:org`, `gist`, `workflow` |

---

*Read from the live system by Hermes, October 2026.*