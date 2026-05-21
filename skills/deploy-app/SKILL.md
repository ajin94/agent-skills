---
name: deploy-app
description: Guide a deployment to a target environment (staging, production, etc.). Use when the user wants to deploy, release, or ship the application.
allowed-tools: Bash(git *) Read
arguments:
  - name: env
    description: Target environment — staging | production | preview (required)
disable-model-invocation: false
---

## Instructions

Deploy to **$env** environment.

### Pre-flight checks (always run these first)
1. Confirm there are no uncommitted changes: `git status`
2. Confirm tests pass before continuing (invoke `run-tests` skill if needed)
3. Confirm the current branch is appropriate for $env:
   - `production` → must be on `main`
   - `staging` → any branch is fine
   - `preview` → any branch is fine

### Deployment steps
Detect the deployment method from project files:

| File present | Method |
|---|---|
| `Dockerfile` | Docker build + push |
| `vercel.json` | `vercel deploy --env $env` |
| `fly.toml` | `fly deploy` |
| `.github/workflows/deploy.yml` | Trigger GitHub Actions |
| `Makefile` with `deploy` target | `make deploy ENV=$env` |

If none match, ask the user which deployment method to use before proceeding.

### Post-deploy
- Report the deployed URL or artifact tag
- Flag any rollback steps if something went wrong
