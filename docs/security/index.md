# Security

Gako is built so the server is a non-target: a full server compromise reveals no
secret content. This page is the operator-facing summary of that posture. For the
conceptual explanation, see the [zero-knowledge model](../concepts/zero-knowledge.md).

!!! warning "Pre-release"
    Gako has **not been audited**. Treat it as experimental and do not store real
    secrets in it yet.

## What the server learns

The server stores opaque ciphertext, access policy, and signatures. From those it
necessarily learns **metadata**: that objects exist and roughly how large they
are (ciphertext is bucketed, not measured exactly), when they are created and
change, and the **access graph** — which identities may read which objects. What
it never learns is the **plaintext** of any secret, because it never holds the
decryption keys.

!!! note "Draft"
    The precise, line-by-line accounting of what the server stores and what it
    can infer lives in the Gako design document, data-model specification, and
    threat-model checks. These will be linked here once the source repository is
    public.

## What Gako does not protect against

Making the server a non-target leaves the endpoints — and the choices operators
and users make — squarely in scope. Zero-knowledge does nothing for:

- **Compromised client devices.** A device compromised while unlocked exposes
  whatever it can already decrypt; endpoint hygiene is outside Gako's reach.
- **Weak or reused master credentials.** A phished, guessed, or reused master
  password defeats the account it protects. Encourage strong, unique passwords,
  and treat recovery codes as offline secrets.
- **Misconfigured access.** Policy is enforced exactly and honestly as
  configured — granting the wrong identity grants the wrong identity. Review
  grants and machine scopes.

The [zero-knowledge model](../concepts/zero-knowledge.md) frames the same
boundary from the user's side.

## Reporting a vulnerability

Report suspected vulnerabilities **privately** — please do not open a public issue
for anything you believe is exploitable. The formal channel is GitHub Security
Advisories on the source repository, which opens to the public once that
repository does; until then, contact the maintainer directly.

What to expect for a confirmed report:

- acknowledgement within 7 days;
- an assessment and remediation plan within 30 days;
- credit in the release notes, unless you would rather not.

There is no bug-bounty program. Several properties are **documented
non-guarantees**, not bugs, so it is worth reading the
[zero-knowledge model](../concepts/zero-knowledge.md) first: the server sees the
access graph, timing, and ciphertext size buckets; revocation cannot recall
values already read; and recovery material is, by design, an offline all-powerful
artifact.
