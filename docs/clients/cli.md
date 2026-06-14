# CLI client

The command-line client shares the same cryptographic core as the web client,
so secrets are interchangeable between them. It is aimed at scripting,
automation, and machine identities.

It builds the same way as the rest of the project:

```sh
cd clients/cli && go build -o gako .
```

!!! note "Draft"
    To document:

    - The session model (signing in, unlocking, session lifetime)
    - Command reference
    - Storing and retrieving secrets from scripts
    - Using machine identities for unattended automation
    - Output formats and exit codes

## Related

- [Web client](web.md)
- [Administration → machine authentication](../admin/index.md)
