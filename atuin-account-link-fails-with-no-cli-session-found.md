---
type: Note
status: Active
related_to: "[[recreating-an-atuin-sync-account-same-username-different-email]]"
---

# atuin account link fails with "No CLI session found"

Gotcha: running `atuin account link` on a machine where the CLI has never logged in fails immediately. This typically happens when the account was **created directly on Atuin Hub** — for that kind of account the command is simply not applicable, and the error is expected.

Verified against `atuin 18.24.3`: `atuin account link --help`, `crates/atuin/src/command/client/account/link.rs`, `crates/atuin-client/src/hub.rs`.

## What `atuin account link` actually does

Its own help line is the precise definition:

> Link your CLI sync account to your Hub account

The direction is **CLI → Hub**, never the reverse. It takes an existing *sync account* — one created from the CLI with `atuin account register` (e.g. on a self-hosted sync server) — and attaches it to your *Hub* account, so the synced records become accessible under Hub auth (unified login, Hub features). Internally it POSTs the CLI session token to `hub/api/v0/account/link`; if the sync account is already linked to a Hub, the server replies `409` and atuin silently treats it as "already linked".

Two inputs it requires:

1. a local **CLI sync session token** in the meta store (created by `atuin account register` or `atuin login` on this machine) — that is the *source* being linked;
2. a **Hub session** — if missing, the command opens an interactive browser login (`/auth/cli` device flow) *during* the link, you do not run it separately.

## The error

```text
$ atuin account link
 ERROR  error=No CLI session found. Please log in first with 'atuin login'.
Error: No CLI session found. Please log in first with 'atuin login'.

Location:
    crates/atuin/src/command/client/account/link.rs:13:9
```

That bail is requirement 1 being checked at the top of `run()`: no local sync session token, so there is no CLI account to use as the source of a link.

## Why a Hub-created account hits this

If the account was **born on Atuin Hub**, its sync account already belongs to Hub — there is nothing to link, in either direction. On a fresh machine the CLI meta store has never held a session, so `account link` fails with exactly this error.

The "log in first" hint in the message is technically the precondition for the command to run at all, **not** the solution for a Hub-created account: even after `atuin login`, `account link` would have nothing new to do (it ends in "already linked"). For a Hub-created account the right answer is to skip the command entirely.

## What to do instead

A Hub-created account only needs the CLI to log in:

```sh
atuin login -u <USERNAME>   # prompts for password, then for the encryption key
atuin sync
```

`atuin account link` is only needed when the sync account was **created from the CLI** and you want it attached to your Hub account.

## Which flow needs which command

| Scenario | Correct flow |
|---|---|
| Sync account registered via `atuin account register`, existing Hub account | `atuin account link` (opens the Hub browser login itself if needed) |
| Account created on Atuin Hub, now using the CLI | `atuin login -u <USER> -k "<KEY>"` — **no link step exists**, it is already a Hub account |
| New machine, existing account of any kind | `atuin login` then `atuin sync` — no link step |
| `atuin account link` before ever logging in locally | Fails: `No CLI session found` (expected — the command needs a CLI-created sync account to link *from*) |

Related: [[recreating-an-atuin-sync-account-same-username-different-email]] — step 4 there registers the account **via CLI**, which is exactly the case where linking to Hub *does* apply.
