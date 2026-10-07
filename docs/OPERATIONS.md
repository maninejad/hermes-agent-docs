# Operations

Runbook for setting up and verifying Hermes + GitHub on a machine like this one.
Derived from what was actually executed, not from theory.

---

## Verify Hermes GitHub access

```bash
export PATH="$HOME/.local/bin:$PATH"   # if gh was installed there

gh auth status
```

Expected output: `✓ Logged in to github.com account <name> (~/.config/gh/hosts.yml)` with
the token scopes listed.

## Install `gh` without root

Needed when the distribution has no `gh` package (true of Kali) and `sudo` is unavailable.

```bash
# 1. find the latest release tag
curl -sL -o /dev/null -w '%{url_effective}\n' https://github.com/cli/cli/releases/latest
# -> https://github.com/cli/cli/releases/tag/vX.Y.Z

# 2. download that release for this architecture
cd /tmp
curl -sL -o gh.tar.gz \
  "https://github.com/cli/cli/releases/download/vX.Y.Z/gh_X.Y.Z_linux_amd64.tar.gz"

# 3. extract just the binary to a user-writable PATH dir
tar xzf gh.tar.gz
mkdir -p ~/.local/bin
cp gh_X.Y.Z_linux_amd64/bin/gh ~/.local/bin/gh
chmod +x ~/.local/bin/gh

# 4. ensure ~/.local/bin is on PATH (usually already in ~/.bashrc)
gh --version
```

> Gotcha: `curl -L .../releases/latest/download/gh_linux_amd64.tar.gz` returns the text
> "Not Found" (9 bytes). The asset name embeds the version, so resolve the tag first.

## Authenticate `gh` headlessly (OAuth device flow)

```bash
# request a device code with gh's public client id
RESP=$(curl -s -X POST -H "Accept: application/json" \
  -d "client_id=178c6fc778ccc68e1d6a&scope=repo,read:org,gist,workflow" \
  https://github.com/login/device/code)
# show the user: verification_uri + user_code, then poll for the token
```

Poll `https://github.com/login/oauth/access_token` with `grant_type=urn:ietf:params:oauth:grant-type:device_code`
until a token comes back. Handle `authorization_pending` (keep polling), `slow_down` (add
5s to the interval), `expired_token` (restart), `access_denied` (stop).

### Pitfall — `gh auth login --with-token` hangs

On headless Linux with no desktop keyring, gh's credential storage blocks indefinitely.
Wrap it in `timeout 20 ...`; on exit 124, write the store directly instead:

```bash
LOGIN=$(curl -s -H "Authorization: token $TOKEN" https://api.github.com/user \
  | python3 -c "import sys,json; print(json.load(sys.stdin)['login'])")
mkdir -p ~/.config/gh
printf 'github.com:\n    users:\n        %s:\n            oauth_token: %s\n    git_protocol: https\n    oauth_token: %s\n    user: %s\n' \
  "$LOGIN" "$TOKEN" "$TOKEN" "$LOGIN" > ~/.config/gh/hosts.yml
chmod 600 ~/.config/gh/hosts.yml
gh auth status          # reads hosts.yml directly, no keyring
gh auth setup-git
```

**Never print the token.** Keep it in a mode-600 file or pipe it straight through; do not
let it appear in a transcript or shell history.

## Create a repository and push

```bash
cd /path/to/content
git init -b main
git add -A
git commit -m "message"
gh repo create <name> --public --source=. --remote=origin --push
```

## Check git credential wiring

```bash
git config --global --get-all credential.helper   # expect: gh auth git-credential
```

If this is empty after `gh auth setup-git`, `gh` commands still work but a bare
`git push` over HTTPS will not. Re-run `gh auth setup-git`, or set the helper explicitly:

```bash
git config --global credential.helper "!gh auth git-credential"
```

---

*Written by Hermes, October 2026.*