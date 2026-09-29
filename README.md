# grid-config

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Part of Grid Platform](https://img.shields.io/badge/Grid%20Platform-open%20source-0A7EA4)](https://github.com/gridplatform)

**Desired-state repository for [Grid Platform](https://github.com/gridplatform)** — the open-source infrastructure orchestration stack.

This repo holds **GitOps intent**: portable Grid JSON that describes what should exist in each cloud and environment. Grid Core, CLI, and Console read it, generate Terraform (and related artifacts) into `archive/`, and apply. If you ever stop using Grid, the committed `archive/` tree remains runnable with plain Terraform.

> **Give back.** Grid exists so teams can own their infrastructure code without vendor lock-in. This repository is the public reference layout and demo catalog the community can fork, learn from, and extend.

## Part of Grid

| Component | Role |
|-----------|------|
| **[grid-config](https://github.com/gridplatform/grid-config)** (this repo) | Desired-state JSON + generated `archive/` |
| **[grid-terraform](https://github.com/gridplatform/grid-terraform)** | Reusable Terraform module bank |
| **[grid-core](https://github.com/gridplatform/grid-core)** | API — syncs this tree, plans/applies, GitOps |
| **[grid-cli](https://github.com/gridplatform/grid-cli)** | Generate / plan / deploy / prune from JSON |
| **[grid-ui](https://github.com/gridplatform/grid-ui)** | Console (projects, environments, releases) |

Website: [gridplatform.org](https://gridplatform.org) · Docs: [grid-docs](https://github.com/gridplatform/grid-docs) · Community: [Discord](https://discord.gg/gridplatform)

## What lives here

- **Intent only under `projects/`** — human-authored Grid JSON.
- **`archive/`** — Terraform Grid generates next to that intent (commit it for an exit path).
- **No product source code** — modules live in `grid-terraform`; the API lives in `grid-core`.

Point Core at this checkout:

```bash
export GRID_CONFIG_ROOT=/path/to/grid-config
# optional remote GitOps (same working tree)
export GRID_GITOPS_REPO_URL=https://github.com/gridplatform/grid-config.git
export GRID_GITOPS_BRANCH=main
```

Or clone and use with the CLI:

```bash
git clone https://github.com/gridplatform/grid-config.git
cd grid-config
grid status --config-dir .
```

## Layout

```text
projects/
  <project-slug>/                 # e.g. demo-app
    .grid/project.json            # optional name / description
    <cloud>/                      # aws | gcp | azure | oracle | …
      <environment>/              # development | staging | production
        <infra-type>/             # vpc, ec2, gke, …
          <resource-name>.json    # one deployable unit

archive/                          # generated Terraform (commit with JSON)
  projects/<project-slug>/<cloud>/<environment>/<infra-type>/<resource-name>/
```

Grid Core discovers **projects only under `projects/<slug>/`**. Add another app by creating a new folder — no Core hardcoding.

### Example paths

```text
projects/demo-app/aws/production/ec2/api-01.json
projects/demo-app/gcp/development/gcs/example-gcs.json
projects/demo-app/alibaba/development/vm/bastion.json   # bastion = a VM named bastion
```

| Segment | Meaning |
|---------|---------|
| `projects/<slug>` | Application / tenancy boundary |
| cloud | Provider |
| environment | Encoded in the path |
| infra-type | Resource type (folder overrides: AWS `vm` → `ec2`, Azure `vm` → `virtual-machine`) |
| `<name>.json` | One unit; filename should match `metadata.name` / primary resource `name` |

### Optional metadata

- `metadata.environment` — should match the path env
- `metadata.modulePath` — module in `grid-terraform` (e.g. `aws/vpc`)
- `metadata.dependsOn` — other JSON paths (relative to this repo) whose vpc/subnet resources merge into generate/plan/deploy

```json
"metadata": {
  "dependsOn": ["projects/demo-app/gcp/development/vpc/grid-development-vpc.json"]
}
```

**Ownership:** the unit you deploy owns the merged graph. Do not also apply the dependsOn VPC unit separately.

## Demo catalog (`projects/demo-app`)

`demo-app` is a **multi-cloud example catalog** (development / staging / production) for exploring Grid. Many files use `REPLACE_*` placeholders — fill those before a real apply.

A bastion host is **not** a separate infra type: it is a normal `vm` (or provider equivalent) **named** `bastion`.

## Placeholders before apply

- Values starting with `REPLACE_` must be filled for real deploys
- GCP `project` / service account fields as required by the module
- AWS / Azure account or subscription labels where noted in examples

## Day-to-day CLI

```bash
grid status --config-dir .
grid deploy --config-dir . --reconcile   # apply added/changed only
grid prune --config-dir .                # list units removed from JSON
grid prune --config-dir . --destroy      # confirm destroy of stale cloud state
```

Deleting a JSON file does **not** destroy cloud resources by itself — the unit becomes **stale** until you restore the JSON or confirm destroy (Console → Infrastructure, or `grid prune --destroy`).

## Contributing

Improvements to the layout, clearer examples, and real-world patterns are welcome.

1. Fork and branch from `main`
2. Keep intent under `projects/<slug>/…`; do not commit secrets
3. Prefer small, named units over giant JSON blobs
4. Open a PR describing the scenario

Issues: [github.com/gridplatform/grid-config/issues](https://github.com/gridplatform/grid-config/issues)

## License

MIT — see [LICENSE](LICENSE). Part of the [Grid Platform](https://github.com/gridplatform) open-source project.

**Built with ❤️ for the community**
