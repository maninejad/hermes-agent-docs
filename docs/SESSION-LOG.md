# Session Log

Chronological record of work done with Hermes in this environment. Newest entries at the
bottom. Each entry states what was asked, what was done, and how it was verified.

---

## 2026-10-07 — Bootstrap: GitHub access, then this repository

**Asked:** "access my github", then "start a repo and document yourself (Hermes and what we
are doing here)."

### Step 1 — Reconnaissance

Checked the environment before acting. Findings:

- `gh` (GitHub CLI) was **not installed**.
- No GitHub token: no `GH_TOKEN`/`GITHUB_TOKEN` environment variable, no `~/.config/gh`,
  no `~/.git-credentials`.
- No SSH key (only an empty `~/.ssh/agent/` directory).
- No global git identity (`user.name` / `user.email` unset).
- The checked-out `hermes-agent` repo points at the public upstream
  `NousResearch/hermes-agent`, not a personal account.

Conclusion: no personal GitHub account was wired up yet.

### Step 2 — Installing the GitHub CLI without root

`sudo` required a password and was unavailable to the agent. On Kali, `apt-get download gh`
failed — **the package does not exist in Kali's repositories**. So `gh` was installed the
portable way:

1. Resolved the latest release tag (`v2.102.0`) via the releases URL.
2. Downloaded `gh_2.102.0_linux_amd64.tar.gz` from GitHub Releases.
3. Extracted just the binary to `~/.local/bin/gh` (already on `PATH` via `~/.bashrc`).

Verified: `gh version 2.102.0 (2026-09-30)`.

### Step 3 — OAuth device-flow login

Used GitHub's OAuth **device flow** with the gh CLI's public client ID, since an interactive
browser login is impractical from an agent session. The user was shown a URL
(`https://github.com/login/device`) and a short code to approve on their own device; the
agent polled for the token.

**Two attempts.** The first polling loop was interrupted before the token exchange
completed — the device code had been approved, but the authorization was never redeemed.
A fresh code was issued and polled in the *second* attempt.

**Key pitfall hit:** `gh auth login --with-token` **hung** (exit code 124 under a 20-second
timeout). This is a known headless-Linux issue: gh's credential storage blocks waiting on a
desktop keyring (secret-service / dbus) that does not exist on this box. The fix is to skip
gh's own login machinery and write the credential store directly:

- Resolved the login name from `https://api.github.com/user`.
- Wrote `~/.config/gh/hosts.yml` by hand (mode 600).
- `gh auth status` and `gh auth setup-git` then read the file store without touching the
  keyring and work immediately.

This worked on the first try once applied.

### Step 4 — Verification

Live API checks, not assumptions:

```
$ gh api user --jq '{login,name,public_repos,followers}'
{"followers":0,"login":"maninejad","name":"Mani Nejad","public_repos":4}

$ gh repo list --limit 5
maninejad/maninejad
maninejad/Active-Directory-Lab
maninejad/aws-cyber-lab
maninejad/zphisher   (fork)
```

Confirmed scopes: `repo`, `read:org`, `gist`, `workflow`.

**Open item noted:** `gh auth setup-git` ran but `git config --global credential.helper`
came back empty, so a bare `git push` over HTTPS would not yet pick up the token. `gh`
commands work fully regardless. Flagged to the user; not yet resolved.

### Step 5 — Creating this repository

Local content was authored first (this README and the `docs/` set), then the repository was
created under the user's account and the content pushed. Repository name:
`maninejad/hermes-agent-docs`.

---

## Conventions established

- **Verify, don't assume.** Every claim about remote state (auth, repos, CI) is backed by a
  fresh command read.
- **Never echo secrets.** OAuth tokens and device codes were handled in files with mode 600
  and never printed into the transcript.
- **Prefer the portable path.** When `sudo` and packaged installs are unavailable, extract a
  release tarball into `~/.local/bin` rather than asking for elevated privileges.

---

*Logged by Hermes, October 2026.*