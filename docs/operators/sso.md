# SSO & Identity

DC Suite is **single sign-on only**. There are no local passwords; every
sign-in goes through an OIDC identity provider (IdP).

## Two sign-in doors

DC Suite exposes two paths:

- **User login** (`/login`) — for people running workloads.
- **Operator/admin login** (`/admin/login`) — gated separately for privileged
  access.

`/connect` redirects into the appropriate flow. Keeping the doors separate
means operator access can be held to a stricter IdP/policy than general user
access.

## Configuring the IdP

Provide the standard OIDC parameters:

- **Issuer URL**
- **Client ID**
- **Client secret** (via a secret, never in plain config)
- Redirect/callback URLs for the console

Once configured, users who exist in the IdP and have been added to an
organization can sign in.

## Per-organization SSO (multi-tenant)

An installation can host multiple organizations, each with its **own** IdP.
The tenancy boundary is the `(issuer, client_id)` pair: a token is matched to
the organization whose IdP issued it. This lets each customer bring their own
identity provider while sharing one DC Suite installation.

## Roles and scopes

Authorization is **scoped RBAC**:

- A **role** grants a set of permissions (e.g. `cluster:write`,
  `billing:write`, `baremetal:read`).
- A **scope** binds the role at **installation**, **organization**, or
  **tenant** level.

A user's effective grants are the union of their role assignments. The API
reports them at `GET /v1/whoami` (`effective_grants`, `installation_admin`).

### Common roles

| Role | Typical grants |
| --- | --- |
| Installation admin | Everything, at installation scope. |
| Org admin | Manage users, clusters, billing currency within an org. |
| Tenant user | Create/run/delete clusters within a tenant. |
| Read-only | View clusters, cost, observability. |

## First-run and account state

A user who has authenticated but not yet been assigned to an organization is
in a **signup** state with no resources — an admin adds them to an org (and a
tenant) to grant access.

## Troubleshooting sign-in

- **Redirect loop or "not authorized"** — the account isn't in an
  organization yet, or the role lacks access to the page.
- **Callback mismatch** — the IdP's redirect URI doesn't match the console's.
- **Operator door refuses a valid user** — operator access is gated
  separately; the user needs an installation/operator grant, not just a user
  grant.

---

**Next:** [Deploys & Releases](deploys.md).
