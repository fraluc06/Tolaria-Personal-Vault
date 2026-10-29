---
type: Runbook
status: Active
url: https://atuin.sh
trigger: "Recreate an atuin sync account keeping the same username but with a new email"
verified_against: "atuin 18.24.x · macOS (zsh)"
---

# Recreating an Atuin sync account (same username, different email)

Checklist for recreating an Atuin sync account with the **same username** but a **new email**, with a fresh encryption key and a clean server-side store.

Tested against Atuin `18.24.x` on macOS (zsh). Flags and output can differ between versions, so check `atuin <command> --help` when in doubt.

## Key facts

- Your **local history** lives in `~/.local/share/atuin/` (`history.db`, `records.db`, `key`). Deleting the server account does **not** delete it.
- `atuin account delete` removes **all synced data on the server**.
- The **daemon** keeps the encryption key in memory. If it is running during a rekey, it keeps writing records encrypted with the *old* key, and `atuin store verify` fails with `bad key`. **Always stop the daemon first.**
- Never paste the output of `atuin key` anywhere public or in a chat. Store it in a password manager.

## 0. Prerequisites

- The old account's username is the one you want to keep. A username can only be reused after the old account has been deleted (step 3).
- If other devices have history that is not on this machine, sync them first (see "Multiple devices" below).

## 1. Stop the daemon and automatic sync

Edit `~/.config/atuin/config.toml`:

```toml
auto_sync = false

[daemon]
enabled = false
```

Then stop anything still running:

```sh
atuin daemon stop
pgrep -fl atuin        # must print nothing
```

If it comes back on its own, it is started by a launchd service or a shell hook: disable it there.

## 2. Back up local data

```sh
cp -r ~/.local/share/atuin ~/atuin-backup
cp -r ~/.config/atuin ~/atuin-config-backup
```

## 3. Delete the old account

While still logged in:

```sh
atuin account delete   # deletes the account and ALL synced data on the server
atuin logout           # if needed
```

## 4. Register the new account

Same username, new email:

```sh
atuin account register -u <USERNAME> -e <NEW_EMAIL>
```

- Do **not** pass `-p <PASSWORD>` on the command line: it ends up in your shell history.
- If you also created the user on Atuin Hub, make sure the CLI is authenticated against it (`atuin login`), and check the link with `atuin account link` if needed. If the account was created on the Hub **before** ever using the CLI, `atuin account link` fails with `No CLI session found` — log in first, see [[atuin-account-link-fails-with-no-cli-session-found]].

## 5. Generate a fresh key (optional)

Make sure the daemon is still stopped (`pgrep -fl atuin` prints nothing), then:

```sh
atuin store rekey      # re-encrypts the local store with a new key
atuin key              # save it in a password manager, don't paste it anywhere
```

## 6. Clean the local store and verify

```sh
atuin store purge      # removes local records that can't be decrypted
atuin store verify     # must say "verified OK"
atuin store rebuild history
atuin store verify     # still OK
```

## 7. Replace the server data

Do **not** run a plain `atuin sync` in between: it could pull old records back.

```sh
atuin store push --force
atuin store verify
atuin store status     # local and remote indexes should match
```

Check `atuin store push --help` first to confirm that `--force` overwrites the remote store in your version.

## 8. Test with a clean copy

Download everything into an empty directory and confirm it decrypts with the current key:

```sh
export ATUIN_CONFIG_DIR=/tmp/atuin-test/config
export ATUIN_DATA_DIR=/tmp/atuin-test/data
mkdir -p "$ATUIN_CONFIG_DIR" "$ATUIN_DATA_DIR"

atuin login            # enter the current key when asked
atuin sync -f
atuin store verify
atuin history list --format "{command}" | wc -l

unset ATUIN_CONFIG_DIR ATUIN_DATA_DIR
rm -rf /tmp/atuin-test
```

Expected: no `WARN ... bad key`, `verify` OK, and a command count close to your real history. Use `atuin info` to confirm the variable names for your version.

## 9. Re-enable daemon and sync

Restore `config.toml`:

```toml
auto_sync = true

[daemon]
enabled = true
```

Open a **new terminal tab** (so the daemon starts with the new key), run a few commands, wait for the sync interval (default 5 minutes), then:

```sh
atuin store verify
atuin sync
atuin store verify
```

Both `verify` runs must stay OK.

## 10. Multiple devices

On every other device:

```sh
atuin daemon stop
atuin logout
atuin login -u <USERNAME> -k "<NEW_KEY>"
atuin sync
atuin daemon start   # or open a new shell
```

If a device has history that is not on the main machine, do this **before** step 3: sync it, or after the migration push its local store from that device.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| `bad key` returns right after `purge` | Daemon running with the old key | `atuin daemon stop`, `pgrep -fl atuin`, start again from a new shell |
| `purge` works but `verify` fails after `sync` | Server still holds records encrypted with the old key | Run step 7 (`push --force`) with the daemon off |
| `first missing record index is N` | Gaps in the local index after a purge | Normal; fix with `rebuild history` then `push --force` |
| Old `encrypted key id` in the error | Records from before the rekey | Purge, then replace the server data |
| Key was pasted somewhere public | Key exposed | Repeat steps 1–7 to rekey and replace the server data |
| `atuin account link` → `No CLI session found` | Account was created on Atuin Hub, the CLI has no session | Run `atuin login` first, then `atuin account link` — see [[atuin-account-link-fails-with-no-cli-session-found]] |

## Quick command summary

```sh
atuin daemon stop && pgrep -fl atuin
cp -r ~/.local/share/atuin ~/atuin-backup
atuin account delete && atuin logout
atuin account register -u <USERNAME> -e <NEW_EMAIL>
atuin store rekey && atuin key
atuin store purge && atuin store verify
atuin store rebuild history
atuin store push --force
atuin store verify && atuin store status
```
