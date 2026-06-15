# Features

What you can do with Gako, and where each capability is documented today. Every
feature is built so the server stays unable to read your secrets — see the
[zero-knowledge model](../concepts/zero-knowledge.md) for the guarantee they all
rest on.

!!! note "This guide is still filling in"
    Dedicated, walkthrough-style feature pages are coming as the workflows
    settle. For now this page maps each capability to the section that already
    covers it.

| Capability | What it is | Where it's documented |
|---|---|---|
| **Secrets** | Store and retrieve passwords, API keys, certificates, and notes, encrypted end to end. | [CLI client → Commands](../clients/cli.md#commands) (the `list` / `get` / `create` / `edit` group); a web-client walkthrough is planned. |
| **Sharing and access** | Grant other identities access to a secret without the server ever seeing its content. | [CLI client → Commands](../clients/cli.md#commands) (`share`, `grants`); team-wide sharing in [Administration → Organizations](../admin/index.md#organizations). |
| **Machine authentication** | Non-human identities for automation and CI — read-only and tightly scoped. | [Administration → Machine identities](../admin/index.md#machine-identities-for-automation). |
| **Recovery** | Recover access after a lost master password, without handing keys to the server. | [Administration → Recovery basics](../admin/index.md#recovery-basics). |

See [Concepts](../concepts/zero-knowledge.md) for the model these features are
built on, and [Security](../security/index.md) for what they do and do not
protect against.
