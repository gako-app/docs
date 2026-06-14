# Security

Gako is built so the server is a non-target: a full server compromise reveals no
secret content. This page is the operator-facing summary of that posture. For the
conceptual explanation, see the [zero-knowledge model](../concepts/zero-knowledge.md).

!!! warning "Pre-release"
    Gako has **not been audited**. Treat it as experimental and do not store real
    secrets in it yet.

## What the server learns

The server stores opaque ciphertext, access policy, and signatures. It can
observe metadata — that objects exist, their sizes, and when they change — but
not the plaintext of any secret, because it never holds the decryption keys.

!!! note "Draft"
    The precise, line-by-line accounting of what the server stores and what it
    can infer lives in the Gako design document, data-model specification, and
    threat-model checks. These will be linked here once the source repository is
    public.

## What Gako does not protect against

- Compromise of a **client device** while unlocked.
- A phished, guessed, or reused **master credential**.
- Access granted to the **wrong identity** — policy is enforced exactly as
  configured.

## Reporting a vulnerability

!!! note "Draft"
    The security contact and disclosure process will be published here, mirroring
    the project's `SECURITY.md`.
