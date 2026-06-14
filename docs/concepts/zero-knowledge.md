# The zero-knowledge model

Gako's central claim is narrow and testable: **the server is cryptographically
incapable of reading the secrets it stores.** This page explains what that means
in practice, what it protects, and — just as importantly — what it does not.

## The promise

Every operation that touches plaintext happens on a client. Before anything
leaves your device, it is encrypted with keys the server never sees. The server
receives, stores, and returns *opaque ciphertext*. It can tell you *that* a
secret exists, who is allowed to fetch it, and when it changed — but not *what*
it contains.

This is a structural property, not a policy. There is no "admin override," no
support tool, and no configuration flag that turns it off, because the server
simply does not hold the keys.

## What a server compromise reveals

The honest way to evaluate a zero-knowledge system is to assume the server is
fully owned by an adversary and ask what they learn. With Gako, a complete
compromise — database, backups, and administrators — yields:

| The attacker gets | The attacker does **not** get |
|---|---|
| Opaque ciphertext blobs | The plaintext of any secret |
| Metadata: object existence, sizes, timestamps | The keys to decrypt anything |
| Access policy and signatures | The ability to forge a client's actions undetectably |

!!! note "Draft"
    This is the user-facing summary. The exact, line-by-line accounting of what
    the server stores and what it can infer lives in the Gako design document
    and data-model specification, and will be linked here once the source
    repository is public. See [Security](../security/index.md) for the current
    operator-facing summary.

## What this does not protect against

Zero-knowledge is a property of the *server*. It does not make endpoints safe.
Your secrets are exposed if:

- a **client device** is compromised while unlocked;
- your **master credential** is phished, guessed, or reused; or
- you grant access to the **wrong person** (policy is enforced honestly, but it
  enforces exactly what you configure).

Gako's job is to make the server a non-target. Protecting the clients and the
credentials remains yours.

## Related reading

- [Architecture](architecture.md) — where each piece of the system runs.
- [Security](../security/index.md) — the operator's threat-model summary.
