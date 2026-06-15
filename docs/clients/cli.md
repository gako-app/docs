# CLI client

`gako` is the command-line client. It embeds the same cryptographic
[core](../concepts/architecture.md) as the web client — encrypting, decrypting,
and signing in-process — so a secret written from the CLI is readable in the
browser and vice versa. It is the client for scripting and automation, and the
**only** client that holds an admin key, so it is also how an instance is
[administered](../admin/index.md).

Until there are packaged releases it is built from source:

```sh
cd clients/cli && go build -o gako .
```

See [Installation → Put the binaries on your PATH](../install/index.md#put-the-binaries-on-your-path)
to run it as `gako` from anywhere.

## Sessions and unlocking

Every command that touches secrets needs your account unlocked, which means your
**master password**: it derives the keys that decrypt your vault, locally, in the
`gako` process. The server never receives it.

By default each command prompts for the password and opens a **throwaway session**
that dies with the process. To avoid re-entering it, start a session once and
export it into your shell:

```sh
gako login            # prints: export GAKO_SESSION=...
eval "$(gako login)"  # ...so eval it to keep the session
```

With `GAKO_SESSION` set, later commands reuse that session without prompting;
`gako logout` ends it. Sessions are time-limited server-side, so an expired one
makes the next command prompt again (exit code `3`).

!!! note "Admin commands always re-prompt"
    `gako admin …` operations ask for the master password every time, even with a
    session active — one entry opens a throwaway session *and* unseals the admin
    key. That friction is deliberate; see
    [Administration](../admin/index.md#the-admin-trust-model).

## Environment

| Variable | Purpose |
|---|---|
| `GAKO_SESSION` | Session key from `gako login` (eval its output). Without it, commands prompt for the master password. |
| `GAKO_PASSWORD` | Master password, for non-interactive automation. Prefer the prompt where you can — an environment variable is more exposed. |
| `GAKO_MACHINE_CREDENTIALS` | Path to a machine credential file. Set it to act as that **machine identity**: read-only, short-lived sessions (see below). |
| `GAKO_CONFIG_DIR` | Override the state directory (default: your OS config directory, under `gako/`). |

The server URL is set once with `--server` on `gako register` or `gako login`
(default `http://127.0.0.1:8347`) and remembered afterward.

## Commands

| Group | Commands | What they do |
|---|---|---|
| **Account** | `register`, `login`, `logout`, `status`, `recover` | Create an account, manage your session, show your identity and fingerprint, recover a lost password. |
| **Secrets** | `list`, `get`, `create`, `edit`, `rm`, `rekey` | Store and retrieve secrets, each addressed by id or title. |
| **Sharing** | `share`, `unshare`, `grants`, `contacts` | Grant another user access to one of your secrets, see who has access, and manage the trust-on-first-use contacts you have pinned. |
| **Automation** | `run`, `tasks`, `generate` | `run` injects secret fields into a child process's environment; `tasks` is the grant-fulfilment queue; `generate` makes a password or passphrase (no login needed). |
| **Administration** | `admin`, `machine`, `org` | Admin credentials, machine identities, and organizations — covered in [Administration](../admin/index.md). |

Run any command with `--help` for its flags. Per-command walkthroughs — storing
and retrieving from scripts, sharing, recovery from the CLI — will be added here
as the workflows settle.

## Output and exit codes

Human-readable results go to **stdout**; prompts, tips, and errors go to
**stderr**, so a pipe captures data cleanly. Adding `--json` switches read
commands to a stable, versioned schema (each document carries a `schemaVersion`)
for scripting.

Exit codes are part of the contract:

| Code | Meaning |
|---|---|
| `0` | Success. |
| `1` | Generic, network, or server error. |
| `2` | Bad invocation (usage error). |
| `3` | Not logged in, locked, or session expired. |
| `4` | No match — or no unique match — for a selector. |
| `5` | Write conflict: re-read and retry. |
| `6` | Security refusal: rollback, bad signature, or a changed key. |

## Machine identities

For unattended consumers — CI, daemons, cron — point `GAKO_MACHINE_CREDENTIALS`
at a credential file provisioned by an administrator. The client then runs as that
machine identity: read-only, scoped to exactly its granted secrets, with
short-lived sessions. The usual pattern is `gako run`, which maps secret fields
into a child process's environment and nothing more. See
[Administration → Machine identities](../admin/index.md#machine-identities-for-automation).

## Related

- [Web client](web.md) — the same core, in the browser.
- [Administration](../admin/index.md) — the CLI as the admin and automation vehicle.
- [Architecture](../concepts/architecture.md) — where the CLI sits in the system.
