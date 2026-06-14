# Architecture

Gako is deliberately small: a single server binary, a shared cryptographic core,
and thin clients. The design pushes every plaintext operation to the edge so the
server can stay ignorant of secret content.

## Components

| Component | Role |
|---|---|
| **Core** | The canonical client crypto/sync logic. One implementation (Go), also compiled to WebAssembly for the browser, so every client enforces identical formats. |
| **Server** | Stores and serves opaque ciphertext, access policy, and signatures over an HTTP API. Also serves the web client. Needs only a data directory. |
| **Web client** | The core compiled to WASM, running in the browser. Encrypts and decrypts locally; the server never sees plaintext. |
| **CLI client** | A command-line client sharing the same core, for scripting, automation, and machine identities. |

## The trust boundary

The line that matters runs between the **clients** (trusted with plaintext and
keys) and the **server** (trusted only to store and serve ciphertext, and to
enforce policy honestly).

```
   ┌─────────────┐        ┌─────────────┐        ┌─────────────┐
   │  Web client │        │ CLI client  │        │  Other apps │
   │  (WASM core)│        │   (core)    │        │  (core)     │
   └──────┬──────┘        └──────┬──────┘        └──────┬──────┘
          │  plaintext stays on this side of the line   │
   ───────┼──────────────────────┼─────────────────────┼────────
          │      opaque ciphertext + policy + signatures│
          └──────────────────────┴─────────────────────┘
                                  │
                          ┌───────┴────────┐
                          │     Server     │
                          │ ciphertext +   │
                          │ policy store   │
                          └───────┬────────┘
                                  │
                          ┌───────┴────────┐
                          │  Data directory │
                          └────────────────┘
```

Because the core is shared, a secret written by the web client is readable by
the CLI and vice versa — the formats are defined once and verified against a
common test-vector corpus.

!!! note "Draft"
    Deeper detail — the API contract, the data model, key hierarchy, and sync
    protocol — lives in the Gako specifications and will be linked here once the
    source repository is public.

## Deployment shape

One static binary serves both the API and the web client and needs nothing but a
data directory. See [Installation](../install/index.md) to run it.
