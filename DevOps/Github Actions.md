## 1. Why do we even need this?

Before CI/CD tools, teams did this manually:

- Developer pushes code
- Someone manually runs tests locally
- Someone manually builds the app
- Someone manually deploys to server (often via FTP/SSH, by hand)

Problems with this:

- Human error (forget to run tests, deploy wrong branch)
- Slow feedback loop (bugs found late)
- No consistency (works on my machine syndrome)
- Doesn't scale — 50 developers can't all manually deploy safely

**GitHub Actions solves this** by automating everything that should happen when something happens in your repo — a push, a PR, a tag, a schedule, an issue comment, etc.

It's GitHub's built-in automation/CI-CD engine. No need for external tools like Jenkins, CircleCI, Travis — it lives right where your code lives.

---

## 2. What is GitHub Actions?

GitHub Actions is an **event-driven automation platform** built into GitHub.

Core idea:

```
EVENT happens on GitHub  -->  WORKFLOW triggers  -->  JOBS run  -->  STEPS execute  -->  Result reported back
```

Example events:

- `push` to a branch
- `pull_request` opened/updated
- `schedule` (cron)
- `workflow_dispatch` (manual trigger button)
- `release` published
- `issue_comment` created

You write YAML files describing what should happen. GitHub then spins up a **runner** (a fresh virtual machine or container), executes your steps on it, and destroys it afterward.

Uses go far beyond CI/CD:

- Run tests on every PR
- Build and push Docker images
- Deploy to AWS/Azure/GCP/Kubernetes
- Auto-label issues/PRs
- Auto-close stale issues
- Send Slack notifications
- Publish npm/PyPI packages
- Security scanning (CodeQL, dependency review)
- Auto-format code and commit back

---

## 3. Where workflow files live

```
your-repo/
└── .github/
    └── workflows/
        ├── ci.yml
        ├── deploy.yml
        ├── lint.yml
        └── nightly-tests.yml
```

Rules:

- Must be inside `.github/workflows/`
- Each file = one **workflow**
- File extension: `.yml` or `.yaml`
- **You can have as many workflow files as you want in one repo.** Each is independent, triggers independently, runs independently.

This is important: large repos commonly have 10-30+ workflow files — one for tests, one for linting, one for security scans, one for each deployment environment (staging/prod), one for release automation, etc. They don't interfere with each other unless you explicitly make them depend on each other.

---

## 4. Anatomy of a workflow file

Every workflow YAML has this skeleton:

```yaml
name: CI Pipeline              # Human-readable name shown in GitHub UI

on:                             # Trigger(s)
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:                           # One or more jobs
  build:
    runs-on: ubuntu-latest      # The runner/VM to use
    steps:                      # Sequential steps inside the job
      - uses: actions/checkout@v4
      - name: Install deps
        run: npm install
      - name: Run tests
        run: npm test
```

Let's break down each block deeply.

### 4.1 `name`

Just cosmetic — shown in the Actions tab UI. Optional, but recommended for clarity when you have many workflows.

### 4.2 `on` — Triggers

This defines **when** the workflow runs. Some common patterns:

```yaml
on: push                     # any push, any branch

on:
  push:
    branches: [main, develop]
    paths: ["src/**"]        # only if files under src/ changed

on:
  pull_request:
    types: [opened, synchronize, reopened]

on:
  schedule:
    - cron: "0 2 * * *"      # every day at 2 AM UTC

on:
  workflow_dispatch:         # manual "Run workflow" button
    inputs:
      environment:
        description: "Deploy target"
        required: true
        default: "staging"
        type: choice
        options: [staging, production]

on:
  release:
    types: [published]

on:
  workflow_call:             # makes this workflow reusable by other workflows
```

You can combine multiple triggers on one workflow:

```yaml
on: [push, pull_request, workflow_dispatch]
```

### 4.3 `jobs`

A workflow contains one or more **jobs**. By default, jobs run in **parallel**, on separate runners, unless you say otherwise.

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps: [...]

  test:
    runs-on: ubuntu-latest
    steps: [...]

  build:
    needs: [lint, test]        # waits for lint AND test to succeed first
    runs-on: ubuntu-latest
    steps: [...]

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps: [...]
```

`needs:` creates a dependency graph. Without it, all jobs fire simultaneously the moment the trigger fires — useful for speed, but you need `needs` when order actually matters (e.g., don't deploy before tests pass).

### 4.4 `runs-on` — the runner

This chooses the VM/OS your steps execute on:

- `ubuntu-latest` (most common, cheapest, fastest)
- `windows-latest`
- `macos-latest`
- `self-hosted` (your own machine/server registered as a runner — common in enterprises for GPU jobs, on-prem access, or cost control)

Each job gets a **completely fresh, isolated VM**. Nothing persists between jobs unless you explicitly pass data (artifacts, cache).

### 4.5 `steps`

Each step does ONE thing. Two kinds:

**a) `run:` — execute shell commands**

```yaml
- name: Install dependencies
  run: npm install

- name: Multi-line script
  run: |
    echo "Building app"
    npm run build
    echo "Build complete"
```

**b) `uses:` — reuse a pre-built Action from the marketplace or your own repo**

```yaml
- uses: actions/checkout@v4          # pulls your repo code onto the runner
- uses: actions/setup-node@v4
  with:
    node-version: "20"
- uses: docker/build-push-action@v5
  with:
    push: true
    tags: myapp:latest
```

`actions/checkout` is almost always the FIRST step in any job — without it, the runner is an empty VM with no access to your code.

`with:` passes inputs to the action, just like function arguments.

---

## 5. How it actually works under the hood

1. GitHub's servers watch for the event you specified (e.g. `git push`).
2. When matched, GitHub schedules the workflow onto a **runner** — either GitHub-hosted (ephemeral cloud VM, destroyed after job ends) or self-hosted (your own persistent machine).
3. The runner downloads a fresh copy of the OS image, installs the Actions Runner agent, and connects back to GitHub.
4. Steps execute top to bottom, sequentially, inside that one VM for that one job.
5. Each `uses:` action is basically a packaged program (JavaScript, Docker container, or composite YAML) that GitHub downloads and executes with your given inputs.
6. Logs stream live back to the Actions tab.
7. On completion, GitHub reports success/failure back to the commit/PR (this is what gives you the green check ✅ or red ✗ on a PR).
8. The VM is destroyed. Nothing survives unless persisted via **artifacts** or **cache**.

### Isolation

Every job = new VM. Two jobs in the same workflow do NOT share filesystem state unless you use:

```yaml
- uses: actions/upload-artifact@v4
  with:
    name: build-output
    path: dist/

# in a later/different job:
- uses: actions/download-artifact@v4
  with:
    name: build-output
```

### Caching (huge for speed in production)

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: npm-${{ hashFiles('package-lock.json') }}
```

This avoids re-downloading dependencies on every run — critical when you run CI hundreds of times a day.

---

## 6. Contexts, expressions, and secrets

GitHub Actions gives you dynamic variables via **contexts**, accessed with `${{ }}` expression syntax.

```yaml
steps:
  - run: echo "Branch is ${{ github.ref }}"
  - run: echo "Actor is ${{ github.actor }}"
  - run: echo "Commit SHA is ${{ github.sha }}"

  - name: Only run on main
    if: github.ref == 'refs/heads/main'
    run: echo "Deploying"
```

Common contexts:

- `github.*` → event metadata (repo, actor, ref, sha, event_name)
- `env.*` → environment variables you define
- `secrets.*` → encrypted secrets stored in repo/org settings
- `matrix.*` → values from a matrix build (see below)
- `steps.<id>.outputs.*` → output from a previous step

### Secrets (critical for production)

Never hardcode API keys/passwords in YAML. Store them in **Settings → Secrets and variables → Actions**, then:

```yaml
- name: Deploy
  env:
    AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
    AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
  run: ./deploy.sh
```

Secrets are masked in logs automatically (shown as `***`).

---

## 7. Matrix builds — testing across many configs at once

Instead of writing repetitive jobs, use a matrix to auto-generate combinations:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20, 22]
        os: [ubuntu-latest, windows-latest]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm test
```

This spins up **6 parallel jobs** (3 node versions × 2 OSes) automatically. Used heavily in library maintenance where you must guarantee compatibility across environments.

---

## 8. Reusable Workflows and Composite Actions

At scale, teams stop copy-pasting YAML and instead centralize logic. Two mechanisms:

### a) Reusable workflows (`workflow_call`)

`.github/workflows/deploy-template.yml`:

```yaml
on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string
    secrets:
      DEPLOY_TOKEN:
        required: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying to ${{ inputs.environment }}"
```

Calling it from another workflow:

```yaml
jobs:
  call-deploy:
    uses: ./.github/workflows/deploy-template.yml
    with:
      environment: production
    secrets:
      DEPLOY_TOKEN: ${{ secrets.PROD_DEPLOY_TOKEN }}
```

This is how large orgs maintain ONE deployment pipeline definition and reuse it across 50+ microservice repos.

### b) Composite actions

A custom local "action" bundling multiple steps into one reusable unit — lives in its own folder with an `action.yml`:

```
.github/actions/setup-project/action.yml
```

```yaml
name: "Setup Project"
runs:
  using: "composite"
  steps:
    - uses: actions/setup-node@v4
      with:
        node-version: "20"
    - run: npm ci
      shell: bash
```

Used like:

```yaml
- uses: ./.github/actions/setup-project
```

---

## 9. Environments and approval gates (production safety)

GitHub supports **Environments** (Settings → Environments) like `staging`, `production`, each with:

- Required reviewers (a human must click "Approve" before deploy proceeds)
- Environment-specific secrets
- Wait timers

```yaml
jobs:
  deploy-prod:
    runs-on: ubuntu-latest
    environment: production        # requires manual approval if configured
    steps:
      - run: ./deploy.sh
```

This is exactly how large orgs prevent an accidental push from auto-deploying to production — the pipeline pauses and waits for a designated approver.

---

## 10. Why this matters hugely for production environments

1. **Consistency** — same steps run every time, no "it worked on my laptop."
2. **Speed** — parallel jobs + caching cut feedback time from hours to minutes.
3. **Safety gates** — required reviews, branch protection rules tied to required checks (a PR literally cannot merge until CI passes).
4. **Auditability** — every deploy is logged: who triggered it, what commit, what time, what approved it.
5. **Rollback friendliness** — deployments are code (YAML), so you can revert a bad pipeline change just like any other commit.
6. **Secret management** — centralized, encrypted, scoped per environment; no keys floating in Slack messages or local `.env` files.
7. **Multi-cloud/tooling agnostic** — same platform can deploy to AWS, GCP, Azure, Kubernetes, bare metal via self-hosted runners.

---

## 11. How large organizations actually use it (real patterns)

- **Monorepo path filtering**: Only run a service's tests if files under that service's folder changed (`paths:` filter), saving massive compute at scale.
- **Org-level reusable workflows**: A central "platform" repo holds standardized CI/CD templates; product teams call them via `workflow_call`, ensuring compliance (security scans, license checks) is baked in everywhere without each team reinventing it.
- **Self-hosted runner fleets**: Companies with GPU-heavy ML workloads or strict data-residency rules run their own runner pools (e.g., on Kubernetes via Actions Runner Controller) instead of GitHub-hosted VMs.
- **Required status checks + branch protection**: `main` branch is configured so PRs cannot merge unless `ci/tests`, `ci/lint`, and `security/codeql` all pass — enforced at the GitHub settings level, not just convention.
- **Progressive deployment pipelines**: `build → test → deploy-staging → smoke-test → manual-approval → deploy-production → post-deploy-verification`, all as one traceable workflow.
- **Scheduled workflows for maintenance**: nightly dependency vulnerability scans (`schedule: cron`), automatic stale-issue closing, automatic changelog generation on release.
- **Matrix testing across SDKs/languages**: e.g., a company shipping a client library tests against 5 language runtimes × 3 OSes = 15 jobs, all from one YAML block.
- **OIDC over long-lived secrets**: Modern enterprises avoid storing cloud credentials at all — they use GitHub's OIDC token exchange to get short-lived AWS/GCP/Azure credentials scoped to that one workflow run, eliminating leaked-key risk entirely.

Example of OIDC-based AWS auth (no static keys stored anywhere):

```yaml
permissions:
  id-token: write
  contents: read

steps:
  - uses: aws-actions/configure-aws-credentials@v4
    with:
      role-to-assume: arn:aws:iam::123456789012:role/github-deploy-role
      aws-region: us-east-1
  - run: aws s3 sync ./dist s3://my-prod-bucket
```

---

## 12. Full realistic example — CI + CD pipeline

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

permissions:
  contents: read

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: "20"
      - run: npm ci
      - run: npm run lint

  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm ci
      - run: npm test

  build:
    needs: [lint, test]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: "20"
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/

  deploy:
    needs: build
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: dist
      - name: Deploy to server
        env:
          DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}
        run: ./scripts/deploy.sh
```

Walkthrough of what happens on a push to `main`:

1. `lint` and `test` (×2 node versions = 3 total jobs) run in parallel.
2. Only if ALL of them succeed does `build` run.
3. `build` compiles the app and uploads the compiled output as an artifact.
4. `deploy` waits for `build`, checks it's on `main`, requires production environment approval (if configured), downloads the artifact, and runs the deploy script using a secret key.

---

## 13. Quick reference cheat-sheet

|Concept|Keyword|Purpose|
|---|---|---|
|Trigger|`on:`|When workflow runs|
|Job dependency|`needs:`|Order/sequencing|
|Runner OS|`runs-on:`|VM to execute on|
|Prebuilt step|`uses:`|Reuse marketplace/local action|
|Custom command|`run:`|Shell command|
|Conditional|`if:`|Skip step/job conditionally|
|Parallel testing|`strategy.matrix`|Multiple configs at once|
|Cross-job data|`upload-artifact` / `download-artifact`|Pass files between jobs|
|Speed optimization|`actions/cache`|Skip redundant installs|
|Secrets|`secrets.*`|Encrypted credentials|
|Manual trigger|`workflow_dispatch`|Run-on-demand button|
|Reusability|`workflow_call`|Call one workflow from another|
|Local reusable steps|composite `action.yml`|Package multiple steps as one action|
|Production safety|`environment:`|Manual approval gates|
|Least-privilege access|`permissions:`|Restrict default token scope|

---

## 14. Key mental model to remember

```
Repo → .github/workflows/*.yml (many independent workflows allowed)
Workflow → triggered by an event
Workflow → made of Jobs (parallel by default, or chained via needs)
Job → runs on a fresh isolated VM (runner)
Job → made of Steps (sequential, run top-to-bottom)
Step → either a shell command (run:) or a packaged action (uses:)
Data does NOT persist across jobs unless passed via artifacts or cache
```

Once this model is internalized, every GitHub Actions YAML file — no matter how complex — is just a variation of this same shape.