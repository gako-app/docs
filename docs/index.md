# Gako documentation

**Zero-knowledge secret management.** The server that stores your secrets cannot
read them — by design, not by promise.

Gako stores passwords, API keys, certificates, and notes encrypted end-to-end.
Every cryptographic operation involving plaintext happens on the client; the
server holds only opaque ciphertext, policy, and signatures. A complete
compromise of the server — its database, its backups, its administrators —
reveals no secret content.

!!! warning "Pre-release"
    Gako is under active development and has **not been audited**. Do not use it
    for real secrets yet. This documentation tracks the current development
    build rather than a tagged release.

## Where to start

These follow the order of the pages that come after — start at the top if Gako
is new to you.

<div class="grid cards" markdown>

-   :material-lock: __Understand the guarantee__

    How the server stays unable to read your secrets.

    [:octicons-arrow-right-24: Zero-knowledge model](concepts/zero-knowledge.md)

-   :material-shield-check: __Security posture__

    What the server learns, and what Gako does not protect against.

    [:octicons-arrow-right-24: Security](security/index.md)

-   :material-server: __Administrators__

    Self-host Gako for yourself or a team. Administration is performed with the
    **CLI client** — it is the only client that holds an admin key.

    [:octicons-arrow-right-24: Installation](install/index.md) ·
    [Administration](admin/index.md)

-   :material-account: __End users__

    Store and retrieve secrets through the web or command line.

    [:octicons-arrow-right-24: Web client](clients/web.md) ·
    [CLI client](clients/cli.md)

</div>

## How Gako is put together

Gako is one static server binary plus a shared cryptographic core that runs in
every client:

- a **server** that stores and serves opaque ciphertext, policy, and signatures;
- a **web client** (the core compiled to WebAssembly), served by that same binary;
- a **CLI client** for scripting, automation, and administration.

The same core enforces the same formats everywhere, so a secret written by one
client is readable by another. See [Architecture](concepts/architecture.md) for
the full picture.

Native **desktop and mobile clients** and **browser extensions** are planned and
will share that same core; until they ship, the web and CLI clients are the two
ways in.
