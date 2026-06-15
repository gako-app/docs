# Installation

Gako runs as a single static server binary that serves both the HTTP API and
the web client. It needs nothing but a data directory — no external database,
no message broker, no runtime dependencies.

!!! warning "Pre-release"
    There is no packaged release yet: no tagged binaries, no container images,
    no system packages. The steps below build from source against the current
    development tree. Gako is also **not audited** — run it for evaluation, not
    for real secrets.

This page takes you from a source checkout to a running server you can reach in
a browser. [Administration](../admin/index.md) then bootstraps the first
administrator and stores the first secret.

## Prerequisites

| Requirement | Version | Notes |
|---|---|---|
| **Go** | 1.26.4 or newer | Builds the server, the CLI, and the WebAssembly core. The module sets `go 1.26.4`; an older toolchain will refuse to build. |
| **Node.js** | Current LTS (verified with 26.x) | Only needed to build the **web client**. The server and CLI build with Go alone. |
| **A C compiler** | — | Not required. The build is **pure Go, no cgo**, so the result is a self-contained static binary. |

Supported build/run platforms are Linux and macOS (the project is developed and
tested on both). Because there is no cgo, the server cross-compiles to other
`GOOS`/`GOARCH` targets in the usual Go way; only Linux and macOS are exercised
today.

Check your toolchain:

```sh
go version      # go1.26.4 or newer
node --version  # v26.x (only if you build the web client)
```

## Get the source

Gako lives in one repository (the `core/`, `server/`, `clients/web/`, and
`clients/cli/` modules are wired together by a `go.work` workspace). Clone it
and work from the repository root:

```sh
git clone <gako-repo-url> gako
cd gako
```

!!! note "Draft"
    The source repository is currently private. Once it is public this page
    will link the exact clone URL; until then, use the checkout you were given.

## Build from source

The build has three independent pieces. Build the **web client first** if you
want the browser UI: its output is embedded into the server binary at
`go build` time, so the order matters.

=== "Full build (with web client)"

    ```sh
    # 1. Web client — compiles the WASM core and writes the built app into
    #    server/internal/webui/dist, where the server embeds it.
    cd clients/web
    npm ci
    npm run build
    cd ../..

    # 2. Server — embeds whatever web build is present, then serves it.
    cd server
    go build -o gako-server ./cmd/gako-server
    cd ..

    # 3. CLI — the admin and automation client (see Administration).
    cd clients/cli
    go build -o gako .
    cd ..
    ```

=== "API-only (skip the web client)"

    ```sh
    # The web build is optional. With no build embedded, the server runs
    # API-only and logs "no web client embedded in this binary".
    cd server && go build -o gako-server ./cmd/gako-server && cd ..
    cd clients/cli && go build -o gako . && cd ..
    ```

`npm run build` rebuilds the WASM core, bundles the Svelte app, and runs a
post-build step that adds Subresource Integrity hashes and a hash manifest. It
writes its output straight into the server module
(`server/internal/webui/dist/`); the server's `//go:embed` directive picks it
up on the next `go build`. Build the web client again whenever it changes, then
rebuild the server to re-embed it.

!!! tip "Web client is optional, but recommended for a first run"
    The browser UI is the friendliest way to confirm an instance works. If you
    only need the API and CLI, the API-only build is smaller and has no Node
    dependency — you can always add the UI later by building the web client and
    rebuilding the server.

## Run the server

```sh
./gako-server --listen 127.0.0.1:8347 --data-dir ./data
```

On start it creates the data directory if needed, opens its database, and logs
(structured JSON, to stderr):

```json
{"level":"INFO","msg":"serving embedded web client"}
{"level":"INFO","msg":"gako server listening","addr":"127.0.0.1:8347","data":"./data"}
```

If you built API-only, the first line reads `no web client embedded in this
binary; serving API only` instead.

### Flags

Every flag has a default; each also reads an environment variable so you can
configure the server without changing its invocation.

| Flag | Env | Default | Meaning |
|---|---|---|---|
| `--listen` | `GAKO_LISTEN` | `:8347` | Address the HTTP server binds. Use `127.0.0.1:8347` to accept only local connections (recommended behind a reverse proxy). |
| `--data-dir` | `GAKO_DATA_DIR` | `./data` | Directory holding the database — the entire server state. Created (mode `0700`) if absent. |
| `--debug` | — | `false` | Verbose (`DEBUG`-level) logging. Off by default. |

The server stops cleanly on `SIGINT`/`SIGTERM`, finishing in-flight requests
within a short grace period.

### Operator subcommands

`gako-server` also has two subcommands that act **directly on the data
directory** and then exit, rather than serving. They are the operator's root of
trust — anyone who can run them already has filesystem access to the data
directory, which sits above any API-level control. Both are covered in
[Administration](../admin/index.md):

| Subcommand | Purpose |
|---|---|
| `gako-server enroll-admin --email <user>` | Mint a single-use token that lets an account enroll the first admin credential. |
| `gako-server recovery-challenge --email <user>` | Mint a single-use account-recovery challenge (the reference server has no mail transport, so the operator relays it). |

Both take `--data-dir` (default `./data`) and `--ttl` (default `1h`). They use
the same database as a running server, so you can run them while the server is
up.

## Transport security

Gako is zero-knowledge — the server never sees plaintext — but that is a
property of the **stored data**, not of the **connection**. The web client is
JavaScript and WASM delivered over HTTP, and the API carries session tokens and
ciphertext. In any real deployment you must terminate TLS in front of the
server: without it, a network attacker can tamper with the delivered web app
(defeating the in-browser cryptography entirely) and read session tokens.

The recommended shape is a TLS-terminating reverse proxy in front of a server
bound to localhost:

```sh
./gako-server --listen 127.0.0.1:8347 --data-dir /var/lib/gako/data
```

=== "Caddy"

    ```caddy
    gako.example.com {
        reverse_proxy 127.0.0.1:8347
    }
    ```

=== "nginx"

    ```nginx
    server {
        listen 443 ssl;
        server_name gako.example.com;

        # ssl_certificate / ssl_certificate_key ...

        location / {
            proxy_pass http://127.0.0.1:8347;
            proxy_set_header Host $host;

            # Server-Sent Events (live sync) must not be buffered.
            proxy_buffering off;
            proxy_read_timeout 1h;
        }
    }
    ```

!!! warning "Do not rewrite the request path"
    Admin operations are authenticated with a signature that covers the request
    path **exactly as the server receives it**, including the `/v1` prefix. A
    proxy that strips or rewrites the path will break every admin operation.
    Pass requests through unchanged — proxy `/` to the server's root, do not
    mount Gako under a sub-path. Live sync uses Server-Sent Events on
    `GET /v1/sync/events`, so the proxy must also stream (not buffer) responses.

The server speaks plain HTTP; certificate management, HTTP→HTTPS redirects, and
HSTS belong to the proxy.

## The data directory

Everything the server persists lives under `--data-dir`:

| File | What it is |
|---|---|
| `gako.db` | The SQLite database — principals, public keys, opaque ciphertext blobs, envelopes, policy, sessions, and audit records. |
| `gako.db-wal`, `gako.db-shm` | SQLite write-ahead-log and shared-memory files (WAL mode). Part of the live database state. |

That directory **is** the instance. It contains only what a zero-knowledge
server is allowed to hold — ciphertext, access policy, signatures, and the
metadata enumerated in the data-model specification — never the keys to decrypt
any secret. Backing it up is therefore safe and sufficient; see
[Administration → Backups and restore](../admin/index.md#backups-and-restore).

The directory is created mode `0700`. Keep it that way: while it holds no
plaintext secrets, it holds session and auth material that deserves the same
care as any server's state.

## First contact

With the server running and the web client embedded, open the listen address in
a browser:

```
http://127.0.0.1:8347/
```

You should get the Gako web client (a single-page app). Behind TLS, use your
proxy's hostname (`https://gako.example.com/`) instead.

To confirm the server is up without a browser, two unauthenticated endpoints
return `200 OK` and nothing else:

```sh
curl -s -o /dev/null -w '%{http_code}\n' http://127.0.0.1:8347/healthz   # 200
curl -s -o /dev/null -w '%{http_code}\n' http://127.0.0.1:8347/readyz    # 200
```

A fresh instance has no accounts yet. The next step — registering the first
account and promoting it to administrator — is in
[Administration](../admin/index.md).

## Next steps

- [Administration](../admin/index.md) — bootstrap the first administrator,
  create users and machine identities, and store your first secret.
- [Web client](../clients/web.md) · [CLI client](../clients/cli.md) — connect
  to your instance.
