# Templates

A **template** (or *stack*) is a packaged piece of software you apply to a
running cluster — a scheduler, an ML framework, a monitoring dashboard, or
your own custom stack. Templates let you go from a bare cluster to a
ready-to-use environment in one step.

## Browsing templates

Open **Templates** in the console. Each card shows an icon, a category
(Scheduler, Orchestration, ML Framework, Monitoring, …), and a short
description. Search and category filters narrow the list.

Templates have a **visibility**:

- **Global / system** — published by an administrator, available to everyone.
- **Private** — added by you, visible only to your account.

## Applying a template

1. Open a template and click **Apply to a cluster** (single node or a full
   cluster are both supported).
2. Choose the target cluster and fill in any inputs the template declares.
3. Apply. The run executes and reports progress; when it completes, the
   software is live on the cluster.

You can watch each run's log from the template's run view.

## Adding your own template

Anyone can add a template; **where it's visible depends on who adds it**:

- If an **admin** adds it, it becomes **global** (available to all accounts).
- If a **user** adds it, it's **private** to that user.

Add a template two ways:

- **Upload a `.zip`** of the template, or
- **Provide a download URL** to a `.zip`.

### Template package format

A template zip contains the module's `*.tf` files. It may optionally include a
`manifest.json` at the root describing metadata (title, description, category,
inputs). If present, the manifest populates the catalog entry; if absent,
sensible defaults are derived from the module.

```
my-template.zip
├── manifest.json        # optional metadata (shallowest one wins)
├── main.tf              # the module
├── variables.tf
└── outputs.tf
```

A `manifest.json` looks like:

```json
{
  "title": "JupyterLab",
  "description": "A JupyterLab server exposed on the cluster.",
  "category": "ML Framework",
  "icon": "🪐",
  "applicable_profiles": ["gpu-l40s", "gpu-hgx-h100"],
  "inputs": [
    {
      "name": "port",
      "type": "number",
      "required": false,
      "default": 8888,
      "description": "Port JupyterLab listens on."
    },
    {
      "name": "conda_env",
      "type": "string",
      "required": true,
      "description": "Name of the conda environment to launch in."
    }
  ]
}
```

| Field | Purpose |
| --- | --- |
| `title` / `description` | Shown on the catalog card and detail page. |
| `category` | Filter bucket (Scheduler, Orchestration, ML Framework, Monitoring). |
| `icon` | Emoji or an image URL (see [Icons](#icons)). |
| `applicable_profiles` | Restrict which hardware profiles the template targets; omit for all. |
| `inputs[]` | Parameters the apply form collects: `name`, `type`, `required`, `default`, `description`. |

### How an apply runs

Applying a template runs its module against the target cluster as a tracked
**run**. The platform injects the cluster's context (node addresses, an access
identity, and platform variables such as the cluster id and public domain) so
the module can wire itself to the running cluster. You watch progress in the
run view; when the run reaches `APPLIED`, the software is live. A run that
declares an exposed web port automatically gets the per-cluster public HTTPS
URL described in [Access & Networking](access-networking.md).

!!! tip "Start from an existing template"
    You can **download the `.zip`** of any globally available template, modify
    it, and re-upload it as your own private template. This is the easiest way
    to build a custom stack.

## Icons

Templates show an icon so they're easy to scan:

- Use an **emoji**, or
- Provide an **image URL** (a CDN link) for a logo.

Icons render in a **fixed, uniform slot** — images are fit inside without
cropping, so every card lines up regardless of the source image's aspect
ratio. Admins can change the icon of any template; users can change the icon
of their own private templates.

## Managing templates

| Action | Who |
| --- | --- |
| Apply to a cluster | Anyone with access to the cluster |
| Download the `.zip` | Anyone (for globally available templates) |
| Change the icon | Admins (any) / owners (their private ones) |
| Delete | Admins (any custom) / owners (their private ones) |

API surface: `GET /v1/stacks`, `GET /v1/stacks/{name}`,
`GET /v1/stacks/{name}/download`, `POST /v1/stacks`,
`PATCH /v1/stacks/{name}`, `DELETE /v1/stacks/{name}`.

---

**Next:** [Images](images.md).
