# What is Hermes Agent?

Hermes Agent is a personal AI agent by [Nous Research](https://nousresearch.com). Its
defining property is that the **same agent core** runs across every surface you use it
from, rather than one product per channel:

- a command-line REPL,
- a messaging gateway (Telegram, Discord, Slack, and roughly twenty other platforms),
- an Ink-based terminal UI,
- an Electron desktop app,
- scheduled jobs (cron) that run on their own.

It learns across sessions through **memory** and **skills**, delegates heavy or
parallelisable work to **subagents**, and can drive a real terminal and a real browser.
Authoritative documentation lives at <https://hermes-agent.nousresearch.com/docs>.

---

## Architecture, in the agent's own words

Two invariants shape almost every design decision in Hermes:

**1. Per-conversation prompt caching is sacred.**
The model provider caches the prefix of a conversation to make repeat calls cheaper and
faster. Anything that mutates past context, swaps the toolset, or rebuilds the system
prompt mid-conversation breaks that cached prefix and multiplies cost. The one sanctioned
exception is context compression. Slash-commands that change system-prompt state therefore
defer to the next session unless explicitly told to apply immediately.

**2. The core is a narrow waist; capability lives at the edges.**
Every model tool is sent on every API call, so a new *core* tool is expensive surface. New
capability is expected to arrive as a CLI command plus a skill, a service-gated tool, or a
plugin — not by growing the core.

### The building blocks

| Concept | What it is |
|---|---|
| **Tools** | The functions the model can call (`terminal`, `read_file`, `web_search`, `browser_exec`, delegated tools, and so on). Sent on every request, hence the high bar. |
| **Skills** | Procedural memory, stored as `SKILL.md` files. Loaded on demand when the task matches. They encode *how* to do a task in this environment — pitfalls, exact commands, proven workflows. |
| **Memory** | A small, always-injected store of durable facts: who the user is, stable environment facts, standing conventions. Budget-limited on purpose. |
| **Subagents** | Isolated agent processes with their own context and toolset, used to parallelise work or to keep large intermediate output out of the parent's context. |
| **Plugins** | Extensions that add providers, memory backends, context engines, and other capability without touching the core. |
| **Cron / scheduler** | Jobs that run on a schedule without a person in the loop. |
| **Gateway** | The layer that serves one or many chat platforms from a single process, profile-aware. |

### The "facade + siblings" pattern

Large files in the codebase are split into a **facade** (public entry points, and the
names other packages import) plus **siblings** named `<stem>_<topic>.py`, each owning one
topic. When looking for code by topic you search the siblings, not the facade. This keeps
any single file small and a given concept in exactly one place.

### Extending it

New capability is added, in order of preference:

1. extend existing code,
2. a CLI command + a skill,
3. a service-gated tool,
4. a plugin,
5. an MCP server,
6. a new core tool — only if it is fundamental and otherwise unreachable.

---

## What the agent can actually do, from here

Demonstrated in this environment:

- **Terminal:** run shell commands, install software without root by extracting release
  tarballs into `~/.local/bin`, inspect processes and files.
- **Files:** read, write, and patch files; search code by regex.
- **Web:** search, and extract page content as markdown.
- **Browser:** drive a real browser for pages that resist plain fetching.
- **GitHub:** fully, via the `gh` CLI — repos, issues, pull requests, reviews, CI.
- **Voice:** text-to-speech output.
- **Scheduling:** cron jobs (noting that in a bare CLI session their output is saved
  locally rather than delivered back into the chat).
- **Delegation:** spawn isolated subagents for parallel or context-heavy work.

---

*Written by Hermes, October 2026.*