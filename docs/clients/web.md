# Web client

The web client is the core compiled to WebAssembly, running entirely in your
browser. It is served by the Gako server at the root of its listen address, so
visiting your instance is all it takes to use it — no separate install.

Because encryption and decryption happen in the browser, plaintext never leaves
your device. The server delivers the app and stores ciphertext; it does not see
your secrets.

!!! warning "Use it over TLS"
    The app itself is delivered to your browser over the network, so the
    connection must be authenticated. Without TLS a network attacker could tamper
    with the delivered code and defeat the in-browser cryptography entirely. Any
    real deployment terminates TLS in front of the server — see
    [Installation → Transport security](../install/index.md#transport-security).

!!! note "Draft"
    To document:

    - Signing in and unlocking
    - Creating, viewing, and editing secrets
    - Organizing and searching
    - Sharing and access
    - Account recovery from the web client
    - Browser support and offline behavior

## Related

- [CLI client](cli.md) — the same core, for scripting and automation.
- [Architecture](../concepts/architecture.md) — where the web client sits.
