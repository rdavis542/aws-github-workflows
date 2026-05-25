# aws-github-workflows

Shared reusable GitHub Actions workflows for all `aws-*` Terraform projects.

## Workflows

| Workflow | Purpose |
|---|---|
| `tf-plan-apply.yml` | Terraform plan (on push/PR) + apply (on workflow_dispatch) |
| `tf-destroy.yml` | Terraform destroy with confirmation gate |
| `tfsec.yml` | tfsec security scanning, uploads SARIF to GitHub Security tab |
| `packer-build.yml` | Packer AMI builds via OIDC |

## Usage

### tf-plan-apply

```yaml
jobs:
  terraform:
    uses: rdavis542/aws-github-workflows/.github/workflows/tf-plan-apply.yml@main
    with:
      aws_region: us-east-2                          # default: us-east-2
      tf_version: latest                             # default: latest
      tf_vars: '{"region":"us-east-2","cidr":"10.0.0.0/16"}'  # optional JSON
      var_file: terraform.tfvars                     # optional, relative to terraform/
    secrets: inherit
```

### tf-destroy

```yaml
on:
  workflow_dispatch:
    inputs:
      confirm_destroy:
        description: 'Type "destroy" to confirm'
        required: true
        type: string
      target_resource:
        description: 'Optional: specific resource to target'
        required: false
        type: string

jobs:
  terraform:
    uses: rdavis542/aws-github-workflows/.github/workflows/tf-destroy.yml@main
    with:
      aws_region: us-east-2
      tf_vars: '{"region":"us-east-2"}'
    secrets: inherit
```

### tfsec

```yaml
jobs:
  security:
    uses: rdavis542/aws-github-workflows/.github/workflows/tfsec.yml@main
    permissions:
      actions: read
      contents: read
      security-events: write
      pull-requests: write
```

### packer-build

```yaml
on:
  workflow_dispatch:
    inputs:
      region: ...
      vpc_name: ...
      build_subnet_name: ...

jobs:
  packer:
    uses: rdavis542/aws-github-workflows/.github/workflows/packer-build.yml@main
    with:
      packer_file: ec2-base.pkr.hcl
    secrets: inherit
```

## Secrets

All workflows use `secrets: inherit` from the calling repo. Required secrets:

| Secret | Used by |
|---|---|
| `AWS_ACCESS_KEY_ID` | tf-plan-apply, tf-destroy |
| `AWS_SECRET_ACCESS_KEY` | tf-plan-apply, tf-destroy |
| `AWS_PACKER_ROLE_ARN` | packer-build (OIDC) |
| `TF_VAR_DB_PASSWORD` | tf-plan-apply, tf-destroy (auroradb only — empty for other projects) |
| `INFRACOST_API_KEY` | tf-plan-apply PR comments (optional — silently skipped if absent) |

## Notes

- All workflows default to `us-east-2` for `aws_region`
- `tf_vars` accepts a JSON object; keys become `TF_VAR_<key>` env vars
- `var_file` and `tf_vars` can be used together (ec2-toolbox uses both)
- Infracost runs on PRs with `continue-on-error: true` — no `INFRACOST_API_KEY` needed for other projects
- `TF_VAR_db_password` is set from `TF_VAR_DB_PASSWORD` secret on every plan/apply/destroy; Terraform silently ignores it for projects that don't declare a `db_password` variable
