---
type: Note
_organized: true
url: https://docs.brew.sh/Formula-Cookbook
---
# Testing Homebrew formula/cask changes locally (CI replica)

Before opening a PR to `homebrew-core` or `homebrew-cask`, replicate locally
what the GitHub Actions runners do. Catches failures early and keeps the PR
diff minimal.

## Common prerequisites

- `brew tap homebrew/core` (and/or `homebrew/cask`) if not already tapped.
- `git checkout <your-pr-branch>` inside the tap repo.
- `HOMEBREW_NO_INSTALL_FROM_API=1` is **mandatory** for every command. Without
  it, brew reads formulae/casks from the JSON API and silently ignores your
  working-copy edits. This is the #1 pitfall.

## Formulae (homebrew-core)

### Manual sequence — minimum CI parity

```sh
brew uninstall --force <formula>
HOMEBREW_NO_INSTALL_FROM_API=1 brew install --build-from-source <formula>
brew test <formula>
brew audit --strict [--new] <formula>      # add --new for a new formula
brew style <formula>                        # or: brew lgtm <formula>
```

- `--build-from-source` matches the CI build job (no bottle reuse).
- `brew test` runs the `test do` block; it must assert real functionality, not
  just `--version` / `--help`.
- `--strict` enables extra audits; `--new` adds new-formula checks.
- `brew lgtm` runs style + audit together.

### brew test-bot — exact CI replica

The `tests.yml` workflow invokes `brew test-bot`. Run the same locally:

```sh
brew test-bot --only-formulae --tap=homebrew/core <formula>
# new formula:
brew test-bot --only-formulae --added-formulae=<f> --testing-formulae=<f>
# fast iteration (skip online checks and dependents):
brew test-bot --only-formulae --skip-online-checks --skip-dependents <formula>
# debug flags: --dry-run -v --fail-fast --local
```

Slower than the manual sequence, but it also tests dependents (the most common
source of CI breakage on version bumps).

### Linux CI via Docker

For Linux-only failures, use the official container:

```sh
docker run --interactive --tty --rm --pull always ghcr.io/homebrew/brew:latest /bin/bash
# inside the container: tap, edit the formula, rerun the manual sequence above
```

## Casks (homebrew-cask)

### Setup: tap symlink (temporary override)

Casks live in a separate tap. Symlink your git checkout so brew uses your
working copy. This is temporary — restore at the end.

```sh
# print current state first (so the restore path is visible)
ls -la "$(brew --repository)/Library/Taps/homebrew/"
brew untap homebrew/cask
ln -s ~/path/to/homebrew-cask "$(brew --repository)/Library/Taps/homebrew/homebrew-cask"
# verify
ls -la "$(brew --repository)/Library/Taps/homebrew/"
```

Restore when done:

```sh
rm "$(brew --repository)/Library/Taps/homebrew/homebrew-cask"
brew tap homebrew/cask
```

### Validation sequence

```sh
brew style --fix <token>
brew audit --cask --online <token>
brew audit --cask --new <token>                                    # new casks only
HOMEBREW_NO_INSTALL_FROM_API=1 brew install --cask <token>         # always by token, never by path
open "/Applications/<AppName>.app"                                 # verify it runs
brew uninstall --cask <token>                                      # session data should persist
HOMEBREW_NO_INSTALL_FROM_API=1 brew uninstall --cask --zap <token> # should require re-login
pgrep -lf <AppName>                                                # empty? if not, add bundle IDs to uninstall quit:
```

### Iterating on the zap stanza

`brew uninstall --zap` reads cached metadata from
`/opt/homebrew/Caskroom/<token>/.metadata/<version>/...`, **not** your working
copy. After editing zap paths you MUST reinstall before re-zapping:

```sh
brew uninstall --cask <token>        # plain uninstall
brew install --cask <token>          # refresh cached metadata
brew uninstall --cask --zap <token>
```

Otherwise the `==> Trashing files:` log silently uses the old stanza.

## Git workflow for the PR

You can't push to `Homebrew/homebrew-core` (or `homebrew-cask`) directly —
PRs always come from your fork. Work directly inside the tap repo so the
files you edit are the same ones brew tests (no copy step to a separate
tap needed, as long as `HOMEBREW_NO_INSTALL_FROM_API=1` is set).

The order is: **fork → add fork as remote → branch → edit → test locally
(`brew test-bot`) → commit → push to fork → open PR.** Never push to the
upstream `Homebrew/*` repo.

### One-time setup

```sh
# 1. Fork on GitHub (once per repo)
gh repo fork Homebrew/homebrew-core --remote=false

# 2. Add your fork as a remote (called "fork") in the local tap repo
git -C "$(brew --repository homebrew/core)" remote add fork https://github.com/<your-user>/homebrew-core.git

# 3. Keep main in sync with upstream (run before each new change)
git -C "$(brew --repository homebrew/core)" checkout main
git -C "$(brew --repository homebrew/core)" pull --rebase
```

Repeat the same for `homebrew-cask` (`gh repo fork Homebrew/homebrew-cask`,
`brew --repository homebrew/cask`) if you work on casks.

### Per-change workflow

```sh
cd "$(brew --repository homebrew/core)"
git checkout main && git pull --rebase           # start from fresh main
git checkout -b foo-1.2.3                        # descriptive branch name
brew edit foo                                    # make your changes

# Test locally BEFORE pushing — this is what the CI runners will do.
# Quick: manual sequence (install --build-from-source, test, audit, style)
HOMEBREW_NO_INSTALL_FROM_API=1 brew install --build-from-source foo
brew test foo
brew audit --strict foo
brew style foo
# Thorough: exact CI replica (also tests dependents)
brew test-bot --only-formulae --skip-online-checks foo

# Only after tests pass: commit and push to YOUR fork (not upstream)
git add Formula/f/foo.rb
git commit -m "foo 1.2.3"                        # first line <=50 chars
git push -u fork foo-1.2.3

# Open the PR from your fork -> upstream main
gh pr create --base main --title "foo 1.2.3" --body-file - <<'EOF'
<filled PR template + one-sentence context>
EOF
```

For casks the flow is identical; just work in
`$(brew --repository homebrew/cask)`, edit `Casks/<letter>/<token>.rb`,
and use the cask validation sequence from the sections above instead of
the formula one.

For version bumps, `brew bump-formula-pr --strict <formula> --version=<ver>`
does the branch + commit + push + PR for you (it forks automatically if you
don't have one yet). Use it whenever the change is just url/sha256/version.
You still owe a local `brew test-bot` run before relying on it, but the
command handles the git/PR mechanics end-to-end.

## Commit & PR hygiene

- One formula/cask per PR, one commit per changed file, minimal diff.
- Never edit `bottle do` blocks (BrewTestBot manages them).
- Check open PRs first to avoid duplicates.
- Commit message first line <=50 chars:
  - Version bump: `foo 1.2.3`
  - New formula: `foo 1.2.3 (new formula)`
  - New cask: `token 1.2.3 (new cask)`
  - Fix/change: `foo: <description>`
- Add `revision 1` only when bottles must be rebuilt (dep changes, binary
  behavior changes) — NOT for cosmetic/style fixes.

## Troubleshooting

- My edits are ignored -> forgot `HOMEBREW_NO_INSTALL_FROM_API=1`.
- Can't reproduce a CI failure -> match the CI env; try the Docker container for Linux.
- Dependent formulae break -> `brew test-bot --skip-online-checks <formula>`.
- `audit --new` fails on repo age -> wait 30 days.
- Cask install by file path fails -> use the token name, never the path.
- zap paths stale after editing -> reinstall before re-zapping (metadata cache).

## References

- Formula Cookbook: https://docs.brew.sh/Formula-Cookbook
- Cask Cookbook: https://docs.brew.sh/Cask-Cookbook
- CONTRIBUTING.md and AGENTS.md in homebrew-core
- `brew test-bot --help`, `man brew`
