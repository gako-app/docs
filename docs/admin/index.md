# Administration

This section is for operators running a Gako instance for themselves or a team.

!!! note "Draft"
    This section is an outline. Each topic below will become its own page as the
    administration workflows are exercised and documented.

## Topics to cover

- **First-run setup** — initializing an instance and bootstrapping the first
  administrator.
- **Users and identities** — inviting users, identity lifecycle, deactivation.
- **Access policy** — how access is granted and what the server enforces.
- **Machine authentication** — issuing and managing non-human (machine)
  identities for automation.
- **Recovery** — the account/secret recovery flow and its operator-facing parts.
- **Backups and restore** — what to back up, and how restore interacts with the
  zero-knowledge model.
- **Operations** — monitoring, logs, and routine maintenance.

## A note on the admin tier

Administration in Gako is built on the same trust boundary as everything else: an
administrator manages identities, policy, and the instance — but, like the
server, an administrator cannot read secret content. See the
[zero-knowledge model](../concepts/zero-knowledge.md).
