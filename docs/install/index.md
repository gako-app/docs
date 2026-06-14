# Installation

Gako runs as a single static server binary that serves both the API and the web
client. It needs nothing but a data directory.

!!! warning "Pre-release"
    There is no packaged release yet. The steps below build from source against
    the current development tree. Packaged binaries and container images are
    planned.

## Build from source

You need a recent Go toolchain and Node.js (to build the web client).

```sh
# 1. Build the web client (compiles the WASM core and embeds it in the server)
cd clients/web && npm ci && npm run build && cd ../..

# 2. Build the server
cd server && go build -o gako-server ./cmd/gako-server

# 3. Run it
./gako-server --listen :8347 --data-dir ./data
```

The web app is served at the root of the listen address. Building the web client
is optional — the server compiles and runs API-only if you skip step 1.

!!! note "Draft"
    To document next:

    - System requirements and supported platforms
    - Running behind a TLS-terminating reverse proxy
    - Configuration reference (flags, environment, data directory layout)
    - Backups and upgrades
    - Container / systemd deployment

## Next steps

- [Administration](../admin/index.md) — first-run setup, users, and policy.
- [Web client](../clients/web.md) · [CLI client](../clients/cli.md) — connect to your instance.
