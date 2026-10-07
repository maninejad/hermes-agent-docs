# Hermes Agent — Working Notes

Documentation of the Hermes Agent and of the work done in this environment.

Maintained by **Hermes** (the agent itself) on behalf of **Mani Nejad**.

---

## What this repository is

This repo is the agent documenting itself: what Hermes is, how it is set up on this
machine, and a running log of what we have actually done together. It is written by the
agent, from first-hand observation of the system it runs on — not from marketing copy.

## Contents

| File | What it covers |
|---|---|
| [`docs/WHAT-IS-HERMES.md`](docs/WHAT-IS-HERMES.md) | What Hermes Agent is, its architecture and capabilities |
| [`docs/THIS-ENVIRONMENT.md`](docs/THIS-ENVIRONMENT.md) | The concrete setup here: host, profile, paths, auth |
| [`docs/SESSION-LOG.md`](docs/SESSION-LOG.md) | Chronological log of what we have done |
| [`docs/OPERATIONS.md`](docs/OPERATIONS.md) | Runbook: how to install, authenticate, verify |

## Quick facts

- **Agent:** Hermes Agent (by Nous Research)
- **Model in use:** `deepseek/deepseek-v4.1-flash` via Nous Portal
- **Host:** Kali GNU/Linux Rolling on WSL2 (Windows Subsystem for Linux)
- **Hermes home:** `/home/mani/.hermes`
- **GitHub account:** [maninejad](https://github.com/maninejad)

---

## Note on provenance

Everything in `docs/` was produced by the agent running on this machine, verified against
live tool output. Where a claim could not be verified, it is marked as such rather than
asserted.