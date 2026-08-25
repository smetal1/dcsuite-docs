# Operators Guide

This guide is for teams who **deploy and run** DC Suite — the control plane
that turns a bare-metal GPU pool into a self-service cloud. It complements the
[User Guide](../user-guide/index.md), which covers using the platform.

## What's here

| Topic | Page |
| --- | --- |
| How the pieces fit together | [Architecture](architecture.md) |
| Installing the control plane | [Installation](installation.md) |
| Configuring backends and behavior | [Configuration](configuration.md) |
| Bringing physical machines into the pool | [Bare-Metal Onboarding](bare-metal.md) |
| Wiring up single sign-on | [SSO & Identity](sso.md) |
| Shipping new versions | [Deploys & Releases](deploys.md) |
| Fixing common incidents | [Runbooks](runbooks.md) |

## Operating principles

- **Fail closed on config.** In production, DC Suite refuses to start without
  the endpoints it needs, so a misconfigured deploy never silently degrades.
- **Fail open at runtime.** If an optional backend (metrics, logs) is
  unreachable, provisioning keeps working — only the affected read path
  degrades.
- **The fleet manager owns the hardware.** DC Suite asks the fleet manager to
  reserve, power, validate, and release machines; it does not poke hardware
  behind the fleet manager's back.
- **Everything is auditable.** Privileged actions are recorded; access is
  scoped by role.

!!! warning "Public docs, generic values"
    Examples here use placeholders like `your-dc-suite.example.com`. Never put
    real hostnames, IPs, tenant IDs, or credentials into public documentation.
