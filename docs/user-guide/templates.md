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
inputs). If present, the manifest is used to populate the catalog entry; if
absent, sensible defaults are derived.

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
