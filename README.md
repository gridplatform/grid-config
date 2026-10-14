# grid-config

**Example** desired-state layout for [Grid Platform](https://github.com/gridplatform).

This repository shows how Grid JSON is shaped and organized (projects, clouds, environments, units). Use it to **learn and copy patterns**.

> **Do not fork this for production.**  
> Create a **new** GitHub repository under your org or account, add your own JSON, and point Grid at that repository (`GRID_GITOPS_REPO_URL`). Own the settings; treat this tree as a reference.

**Docs:** [Concepts](https://github.com/gridplatform/grid-docs/blob/main/docs/concepts/overview.md) · [Install](https://github.com/gridplatform/grid-docs/tree/main/docs/install)

## What this is for

| Use | Don’t use for |
|-----|----------------|
| Learning the JSON layout | Your company’s real environments |
| Trying demos with `REPLACE_*` filled in | Long-term GitOps source of truth |
| Opening PRs that improve examples | Fork-and-forget “my infra” |

## Layout

```text
projects/
  <project-slug>/
    <cloud>/                      # aws | gcp | …
      <environment>/              # development | staging | production
        <infra-type>/             # vpc, ec2, …
          <resource-name>.json    # one deployable unit
```

Optional generated Terraform may appear under `archive/` next to intent — useful as a plain-Terraform exit path when you generate locally.

### Example paths

```text
projects/demo-app/aws/production/vpc/example-vpc.json
projects/demo-app/gcp/development/network/example-network.json
```

| Segment | Meaning |
|---------|---------|
| `projects/<slug>` | Application / boundary |
| cloud | Provider |
| environment | Encoded in the path |
| infra-type | Resource type folder |
| `<name>.json` | One unit |

`metadata.dependsOn` lists other unit paths **relative to the project** (reference-only — separate workspaces, linked via remote state).

## Trying the samples

```bash
git clone https://github.com/gridplatform/grid-config.git
cd grid-config
# Prefer copying into YOUR new repository, then:
grid status --config-dir .
```

Fill any `REPLACE_*` values before a real apply. Many units are examples only.

## Related repositories

| Repository | What it’s for |
|------------|---------------|
| [grid-core](https://github.com/gridplatform/grid-core) | Runs Grid; syncs **your** desired-state repository |
| [grid-cli](https://github.com/gridplatform/grid-cli) | Generate / plan / apply from JSON |
| [grid-terraform](https://github.com/gridplatform/grid-terraform) | Modules those JSON files point at |
| [grid-docs](https://github.com/gridplatform/grid-docs) | Guides |

## Contributing

Improvements to examples and clearer layouts are welcome. Keep secrets out of JSON. Prefer small, named units.

Issues: [grid-config/issues](https://github.com/gridplatform/grid-config/issues)

## License

MIT — see [LICENSE](LICENSE).
