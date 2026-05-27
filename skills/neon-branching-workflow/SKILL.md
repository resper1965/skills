---
name: neon-branching-workflow
description: "Manage database branches on Neon Postgres. Covers branching per feature/PR, Vercel preview databases, schema migrations in preview branches, resetting branches, and GitHub Actions integration."
risk: safe
source: custom
date_added: "2026-05-27"
tags: ['neon', 'database', 'branching', 'ci-cd', 'vercel', 'migrations']
tools: ["claude", "cursor", "gemini", "antigravity"]
---

# Neon Database Branching Workflow

This skill guides the implementation of a modern, Git-like database branching workflow using Neon serverless Postgres.

## When to Use This Skill
- Setting up a development or testing environment where each developer or PR needs an isolated database branch.
- Integrating database migrations into CI/CD pipelines (GitHub Actions, Vercel Previews).
- Testing schema migrations on a copy of production data without risk.
- Resetting development databases to a clean parent state.

## Core Operations

### 1. Create a Branch per Feature / PR
Neon allows instant, cost-effective database branching using copy-on-write storage.
```bash
# Create a branch from 'main'
neonctl branches create --name preview/pr-42 --parent main
```

### 2. Retrieve Connection String
```bash
# Get connection string for the new branch
neonctl connection-string preview/pr-42
```

### 3. CI/CD Integration (GitHub Actions + Vercel preview)
In your PR workflow, spin up a database branch, deploy migrations, and inject the connection string into the Vercel preview environment.
```yaml
# .github/workflows/preview.yml
jobs:
  create-preview:
    runs-on: ubuntu-latest
    steps:
      - name: Install Neon CLI
        run: npm install -g @neondatabase/neonctl
      
      - name: Create Database Branch
        id: create-branch
        run: |
          BRANCH_NAME="preview/pr-${{ github.event.number }}"
          neonctl branches create --name $BRANCH_NAME --parent main --token ${{ secrets.NEON_API_KEY }} --project-id ${{ secrets.NEON_PROJECT_ID }}
          CONN_STR=$(neonctl connection-string $BRANCH_NAME --token ${{ secrets.NEON_API_KEY }} --project-id ${{ secrets.NEON_PROJECT_ID }})
          echo "db_url=$CONN_STR" >> $GITHUB_OUTPUT
```

### 4. Running Migrations on Preview Branches
Before deploying code, apply migrations using your ORM (e.g. Drizzle or Prisma) to the isolated branch.
```bash
# Drizzle example
DATABASE_URL=$CONN_STR drizzle-kit migrate

# Prisma example
DATABASE_URL=$CONN_STR prisma migrate deploy
```

### 5. Cleaning Up (Delete Branch)
Once a PR is merged or closed, delete the branch to keep the project clean.
```bash
neonctl branches delete preview/pr-42 --token ${{ secrets.NEON_API_KEY }} --project-id ${{ secrets.NEON_PROJECT_ID }}
```

### 6. Resetting a Branch to Parent State
If a development branch gets corrupted or needs fresh production data:
```bash
# Reset branch preview/pr-42 to main
neonctl branches reset preview/pr-42 --parent main
```
