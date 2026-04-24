# ECR Lifecycle Policy Automation

Automates applying AWS ECR image retention policies across multiple repositories using GitHub Actions. Includes a dry-run preview before any images are expired.

## How it works

1. **Dry run** — previews which images would be expired on the first 3 repos in `repos.txt`
2. **Apply** — applies the policy to every repo in `repos.txt`
3. **Verify** — confirms every repo has a policy attached

The apply job requires manual approval via a GitHub Environment (`production`) before it runs.

## Retention policy

| Image type | Retention |
|---|---|
| Untagged | Expire after 1 day |
| `prod-*` tagged | Keep last 20 |
| `uat-*` tagged | Keep last 10 |
| `dev-*` tagged | Keep last 5 |
| Any other tagged | Keep last 5 |

Modify `lifecycle-policy.json` to change these rules.

## Setup

### 1. Add your repos

Edit `repos.txt` and add one ECR repository name per line:

```
my-backend-service
my-frontend-app
my-worker-service
```

### 2. Add GitHub secrets

In your repo go to **Settings → Secrets and variables → Actions** and add:

| Secret | Value |
|---|---|
| `AWS_ACCESS_KEY_ID` | Your IAM user access key |
| `AWS_SECRET_ACCESS_KEY` | Your IAM user secret key |

### 3. Set the AWS region

In `.github/workflows/apply-ecr-lifecycle.yml`, update the region:

```yaml
env:
  AWS_REGION: us-east-2  # change to your region
```

### 4. Configure the production environment

Go to **Settings → Environments → New environment**, name it `production`, and add required reviewers. This gates the apply job behind a manual approval.

### 5. Attach an IAM policy to your user

The IAM user needs these ECR permissions. See `iam-policy.json` for the policy — replace `<YOUR_AWS_ACCOUNT_ID>` with your account ID before applying.

```json
{
  "Action": [
    "ecr:PutLifecyclePolicy",
    "ecr:GetLifecyclePolicy",
    "ecr:DeleteLifecyclePolicy",
    "ecr:StartLifecyclePolicyPreview",
    "ecr:GetLifecyclePolicyPreview",
    "ecr:DescribeRepositories"
  ]
}
```

## Triggering the workflow

**Automatic** — push a change to `lifecycle-policy.json` or `repos.txt` on `main`. The dry run runs first; apply runs after approval.

**Manual** — go to **Actions → Apply ECR Lifecycle Policies → Run workflow** and choose `dry-run` or `apply`.

## Running locally

Preview what would be expired for a single repo:

```bash
export AWS_REGION=us-east-2
./dry-run.sh my-backend-service
```

Apply to all repos:

```bash
./apply-lifecycle-policy.sh
```

Verify all repos have a policy:

```bash
./verify.sh
```

## File reference

| File | Purpose |
|---|---|
| `lifecycle-policy.json` | The ECR lifecycle policy applied to all repos |
| `repos.txt` | List of ECR repository names to target |
| `dry-run.sh` | Preview expiry for a single repo |
| `apply-lifecycle-policy.sh` | Apply policy to all repos in `repos.txt` |
| `verify.sh` | Confirm every repo has a policy |
| `iam-policy.json` | IAM policy template for the automation user |
