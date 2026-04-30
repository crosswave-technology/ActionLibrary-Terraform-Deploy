# Tech Doc - ActionLibrary-Terraform-Deploy

## Design

Composite action that wraps `ActionLibrary-Terraform-Control` to run Terraform plan
or apply for a single AWS account. Handles:

1. **Git auth** - Configures `git config url.insteadOf` so `terraform init` can clone private module repos
2. **AWS credentials** - Assumes the appropriate OIDC role (`GitHub-Terraform-Plan` or `GitHub-Terraform-Apply`) via `aws-actions/configure-aws-credentials@v4`
3. **Plan flow** - Runs plan with standard S3 backend config, uploads plan artifact for apply stage
4. **Apply flow** - Downloads plan artifact, runs apply against the saved plan

## Backend Configuration

State is stored in S3 with the path:

```
state/repository/{repository_id}/{account_group}/{account_id}/terraform.tfstate
```

Encryption is enforced via the provided KMS key. All backend parameters are caller-supplied — no hardcoded values.

## OIDC Roles

| Command | IAM Role (default) |
|---------|-------------------|
| plan    | `GitHub-Terraform-Plan` |
| apply   | `GitHub-Terraform-Apply` |

Both are overridable via `plan_role_name` / `apply_role_name` inputs.

## External Actions

- `aws-actions/configure-aws-credentials@v4`
- `actions/upload-artifact@v4` / `actions/download-artifact@v4`
- `crosswave-technology/ActionLibrary-Terraform-Control@v1.0.0`