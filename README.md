# Terraform Deploy Reusable Workflow

This repository contains a reusable GitHub Actions workflow for deploying Terraform configurations to AWS using OpenID Connect (OIDC) authentication and GitHub Actions.

It enables any repo in your organization to reuse a shared, secure Terraform CI/CD pipeline.

---

## 🔧 Features

- Deploys Terraform using a shared pipeline
- Supports OIDC authentication to AWS (no long-lived secrets)
- Supports private Terraform modules via GitHub PAT
- Supports working directories (e.g., `terraform/app-layer`)
- Easily invoked by other repos

---

## 📂 File Structure

.github/
└── workflows/
└── terraform-deploy-template.yml


---

## 📥 Inputs

| Name              | Required | Description                                      |
|-------------------|----------|--------------------------------------------------|
| `role-to-assume`  | ✅       | IAM role ARN to assume using OIDC                |
| `aws-region`      | ✅       | AWS region to deploy into (e.g. `eu-west-2`)     |
| `working-directory` | ❌     | Path to the Terraform configuration (default: `.`) |

---

## 🔐 Required Secrets

| Name            | Description                               |
|------------------|-------------------------------------------|
| `TF_GITHUB_PAT` | GitHub Personal Access Token with `repo` scope to access private module repositories |

---

## 🚀 Example Usage in Consuming Repo

Create `.github/workflows/deploy.yml` in your repo:

```yaml
name: Deploy Terraform

on:
  push:
    branches: [main]
    paths:
      - 'terraform/app-layer/**/*.tf'  # optional: only run when .tf files change

permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    uses: ensembleanalytics/ensemble-ci-templates/.github/workflows/terraform-deploy-template.yml@main
    with:
      role-to-assume: arn:aws:iam::123456789012:role/github-access-role
      aws-region: eu-west-2
      working-directory: terraform/app-layer
    secrets:
      TF_GITHUB_PAT: ${{ secrets.TF_GITHUB_PAT }}
```

Replace ensembleanalytics, aws-region, and working-directory with your own values as needed.

## Setup Requirements

### In the template repo:

- This file must live in .github/workflows/terraform-deploy-template.yml
- Marked with on: workflow_call

### In AWS:

- IAM Role (e.g. github-access-role) with:
- Trust policy for GitHub OIDC
- Permissions for S3, DynamoDB, Lambda, etc.

### In GitHub:

- Define TF_GITHUB_PAT secret in each consuming repo or use an org-level secret

### Tips

- Use Git tags (e.g., @v1.0.0) for versioned template usage
- Abstract env-specific variables via inputs (optional)
- Add plan-only or var-file support as needed
