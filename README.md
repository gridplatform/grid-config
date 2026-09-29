# grid-config

Desired-state Grid JSON for platform infrastructure.

Grid reads this intent, generates Terraform / GitOps / Crossplane artifacts under
`archive/`, and applies them. Commit `archive/` with the JSON so the stack stays
operable if Grid is removed (run Terraform from `archive/…`).

This repo is **customer desired state only** — no internal tooling, indexes, or
generator scripts.

## Layout

```text
<cloud>/                      # aws | gcp | azure | …
  <environment>/              # development | staging | production
    <infra-type>/             # provider resource type
      <resource-name>.json    # intent (one deployable unit)

archive/                      # Terraform buffer (grid generate) — commit this
  <cloud>/<environment>/<infra-type>/<resource-name>/
    main.tf
    modules/                  # full copy from grid-terraform
```

`grid generate` / `plan` / `deploy` write under **`archive/`**, mirroring the JSON
path. Same rule for `demo-infra`, this repo, or any `grid init` root.

Examples:

```text
aws/production/ec2/
  api-01.json
  worker-17.json

gcp/development/gcs/
  example-gcs.json

aws/development/s3/
  example-s3.json
```

100 EC2s ⇒ 100 JSON files under `…/ec2/`. Same rule for every infra type.

| Segment | Meaning |
|---------|---------|
| cloud | Provider |
| environment | Env encoded by path |
| infra-type | Provider resource type (folder overrides: AWS `vm` → `ec2`, Azure `vm` → `virtual-machine`) |
| `<name>.json` | That one unit’s Grid config |

Filename should match the primary resource `name` / `metadata.name`.

Optional on each file:

- `metadata.environment` — should match the path env
- `metadata.modulePath` — Terraform / Crossplane module this unit maps to
- `metadata.dependsOn` — paths (relative to this repo root) of other Grid JSON
  files whose **vpc/subnet** resources should be merged for generate/plan/deploy
  (needed for split vpc + vm/ec2 files)

### dependsOn / split vpc + vm

Split vpc + vm JSON files can declare:

```json
"metadata": {
  "dependsOn": ["gcp/development/vpc/grid-development-vpc.json"]
}
```

Pass `--config-dir .` so the CLI merges the dependency’s vpc/subnet resources into
the same generate graph (catalog `foldInto` needs parent + children together).

**Ownership rule:** the unit you deploy (e.g. the VM JSON) will manage the
merged network resources in **its** Terraform workspace. Do **not** also
`deploy` / `--reconcile` the dependsOn VPC unit separately — that creates
duplicate ownership. Prefer either:

1. Deploy only the leaf (VM/EC2) and let dependsOn pull network in, or
2. Keep network + compute in one JSON file.

Cross-stack refs via remote state (apply VPC alone, then VM against its outputs)
are out of scope for this layout; use dependsOn merge or a single JSON unit.

## Placeholders before apply

- GCP `project`: `REPLACE_GCP_PROJECT_<ENV>`
- AWS / Azure `project`: account/subscription label placeholders
- Values starting with `REPLACE_` must be filled for real deploys
- GCP VMs need `metadata.serviceAccountEmail` (or `GRID_GCP_SERVICE_ACCOUNT_EMAIL`)

## CLI: changes vs deletes

This repo is the source of truth. With grid-cli:

```bash
grid status --config-dir .
grid deploy --config-dir . --reconcile   # apply added/changed only
grid prune --config-dir .                # list stale (JSON removed)
grid prune --config-dir . --destroy      # confirm, then destroy
```

`--reconcile` skips JSON units that appear in another unit's `metadata.dependsOn`
(so a VPC covered by a VM/EC2 stack is not applied twice).

Deleting a JSON does **not** auto-destroy cloud resources — it becomes **stale** until you confirm destroy.
