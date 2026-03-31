---
name: github-actions
description: '**WORKFLOW SKILL** — Create, manage, and debug GitHub Actions workflows and custom actions. USE FOR: setting up CI/CD pipelines, integrating with Vault for secrets, creating reusable actions, debugging workflow failures. DO NOT USE FOR: general coding, non-GitHub CI/CD.'
---

# GitHub Actions Workflow Management

## Overview
This skill helps you create, configure, and troubleshoot GitHub Actions workflows for CI/CD pipelines, including integration with Vault for secret management.

## Key Concepts
- **Workflows**: YAML files in `.github/workflows/` that define CI/CD processes
- **Actions**: Reusable components (Docker containers or JavaScript) for common tasks
- **Jobs**: Sets of steps that run on runners
- **Secrets**: Encrypted environment variables for sensitive data

## Common Patterns

### 1. Basic CI Workflow
```yaml
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '18'
      - run: npm ci
      - run: npm test
```

### 2. Vault Integration
For secure secret management, use Vault with custom actions or direct CLI calls.

#### Custom Vault Action Template
```yaml
name: 'Vault Login'
description: 'Authenticate with Vault and retrieve secrets'
runs:
  using: 'composite'
  steps:
    - name: Authenticate
      run: vault login ${{ inputs.token }}
      shell: bash
    - name: Retrieve secrets
      run: |
        # Use vault CLI to get secrets and set environment variables
        echo "SECRET=$(vault kv get -field=value secret/path)" >> $GITHUB_ENV
      shell: bash
```

#### Using Consul Template for .env Generation
```bash
# Template file (env-template.ctmpl)
{{ with secret "secret/path" }}
SECRET_KEY={{ .Data.data.secret_key }}
{{ end }}

# Render command
consul-template -template env-template.ctmpl:.env
```

## Troubleshooting Common Issues

### Action Not Found
- Ensure `action.yml` exists in the action directory
- Check path references (local: `./action-name`, remote: `owner/repo@ref`)
- Verify repository and branch exist

### Vault Connection Issues
- Check VAULT_ADDR environment variable
- Verify token has correct permissions
- Ensure Vault server is accessible from runner

### Environment Variables Not Set
- Use `>> $GITHUB_ENV` to persist variables between steps
- For multi-line, use `EOF` syntax
- Check variable names match exactly

### Workflow Permissions
- Add `permissions:` block for repository access
- Use `contents: read` for checkout
- Add `id-token: write` for OIDC

## Best Practices
- Use reusable workflows for common patterns
- Store secrets in GitHub Secrets or Vault
- Test workflows on feature branches
- Use matrix builds for multiple environments
- Cache dependencies to speed up builds

## Quick Start
1. Create `.github/workflows/` directory
2. Add workflow YAML file
3. Test with manual dispatch
4. Add triggers (push, PR, schedule)
5. Integrate secrets management