- Category: DevOps
- Difficulty: Intermediate
- Related: version-control, es-modules

### CI/CD — Automate Building, Testing, and Deploying Code
CI/CD stands for **Continuous Integration** and **Continuous Delivery/Deployment**. It is a set of practices that automates the journey from writing code to running it in production. Every time a developer pushes code, CI/CD pipelines automatically lint, test, build, and deploy the application.

**Analogy**
A car assembly line. A worker (developer) delivers a new part (code). The assembly line (pipeline) automatically checks the part's quality (lint + test), assembles it into the car (build), and drives the car off the lot to the showroom (deploy). No one has to manually carry the car — the line handles it every single time.

---

### 1. Continuous Integration (CI) — Catch Bugs Early

**Theory**
CI is the practice of frequently merging code changes to a shared branch, with each merge triggering an automated pipeline that validates the code. The goal: find integration bugs immediately, before they stack up into a "merge nightmare" at the end of a sprint.

CI typically runs: **lint → unit tests → build → report**

**Working Flow**

![flow-chart](flow-chart.png)

**Example**
```yaml
# .github/workflows/ci.yml — runs on every push and PR
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Run tests
        run: npm test -- --coverage

      - name: Build
        run: npm run build
```

**Output**
```
✓ Checkout code          (2s)
✓ Setup Node.js          (5s)
✓ Install dependencies   (18s)
✓ Lint                   (4s)  — ESLint passes
✓ Run tests              (12s) — 47 tests passed, 0 failed
✓ Build                  (22s) — dist/ folder generated
All checks passed ✅
```

---

### 2. Continuous Delivery vs Continuous Deployment

**Theory**
These two terms are often confused:
- **Continuous Delivery** — code is automatically built and tested and is always in a deployable state. A human presses the deploy button.
- **Continuous Deployment** — every passing pipeline is automatically deployed to production. No human approval needed.

**Working Flow**

![flow-chart-2](flow-chart-2.png)

**Example**
```yaml
# Continuous Delivery — deploys to staging, waits for manual approval to production
jobs:
  deploy-staging:
    needs: build-and-test
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - name: Deploy to staging
        run: npm run deploy:staging

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://myapp.com
    # 'manual' environment requires a human to approve in GitHub UI
    steps:
      - name: Deploy to production
        run: npm run deploy:production
```

**Output**
```
✓ build-and-test   → passed
✓ deploy-staging   → deployed to staging.myapp.com
⏸ deploy-production → waiting for manual approval...

[Human reviews staging, clicks "Approve" in GitHub]

✓ deploy-production → deployed to myapp.com
```

---

### 3. Pipeline Stages — Full CI/CD Flow

**Theory**
A complete pipeline has distinct stages. Each stage must pass before the next begins. If any stage fails, the pipeline stops and the team is notified.

**Working Flow**

![flow-chart-3](flow-chart-3.png)

**Example**
```yaml
# Full pipeline with caching and environment variables
name: Full CI/CD Pipeline

on:
  push:
    branches: [main]

env:
  NODE_ENV: production
  VITE_API_URL: ${{ secrets.API_URL }}

jobs:
  # Stage 1 — Code quality
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'npm' }
      - run: npm ci
      - run: npm run lint
      - run: npm run type-check

  # Stage 2 — Tests (runs parallel to lint)
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'npm' }
      - run: npm ci
      - run: npm test -- --coverage --watchAll=false

  # Stage 3 — Build (only if lint + test pass)
  build:
    needs: [lint, test]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'npm' }
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: build-output
          path: dist/

  # Stage 4 — Deploy
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
        with: { name: build-output, path: dist/ }
      - name: Deploy to Vercel
        run: npx vercel --prod --token=${{ secrets.VERCEL_TOKEN }}
```

**Output**
```
Stage 1 — Lint + TypeCheck  ✓ 8s
Stage 2 — Tests             ✓ 15s  (parallel with Stage 1)
Stage 3 — Build             ✓ 22s  (only after 1+2 pass)
Stage 4 — Deploy            ✓ 12s  → myapp.vercel.app live
```

---

### 4. GitHub Actions — Core Concepts

**Theory**
GitHub Actions is the most widely used CI/CD platform. Key concepts:
- **Workflow** — a YAML file in `.github/workflows/`
- **Trigger (on:)** — what event starts the workflow (push, PR, schedule, manual)
- **Job** — a group of steps that runs on one machine
- **Step** — a single command or action
- **Action** — a reusable unit (`uses: actions/checkout@v4`)
- **Runner** — the machine that executes the job (`ubuntu-latest`)
- **Secrets** — encrypted environment variables stored in GitHub settings

**Working Flow**

![flow-chart-4](flow-chart-4.png)

**Example**
```yaml
# Trigger options
on:
  push:                        # on git push
    branches: [main, 'feat/*']
  pull_request:                # on PR open/update
    branches: [main]
  schedule:                    # cron — runs nightly at 2am UTC
    - cron: '0 2 * * *'
  workflow_dispatch:           # manual trigger button in GitHub UI

# Matrix strategy — test on multiple Node versions
jobs:
  test:
    strategy:
      matrix:
        node: [18, 20, 22]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
      - run: npm ci && npm test
```

**Output**
```
Triggered by: push to main

3 parallel jobs (matrix):
  test (node-18) ✓
  test (node-20) ✓
  test (node-22) ✓

All matrix jobs passed
```

---

### 5. Real-World — React App CI/CD to Vercel

**Working Flow**

![flow-chart-5](flow-chart-5.png)

**Example**
```yaml
# .github/workflows/deploy.yml
name: Deploy React App

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  ci:
    name: Lint, Test & Build
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node 20
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install
        run: npm ci

      - name: Lint (ESLint)
        run: npm run lint

      - name: Test (Vitest)
        run: npm run test:ci

      - name: Build (Vite)
        run: npm run build
        env:
          VITE_API_URL: ${{ secrets.VITE_API_URL }}

  deploy:
    name: Deploy to Vercel
    needs: ci
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Deploy
        run: |
          npm i -g vercel
          vercel pull --yes --environment=production --token=${{ secrets.VERCEL_TOKEN }}
          vercel build --prod --token=${{ secrets.VERCEL_TOKEN }}
          vercel deploy --prebuilt --prod --token=${{ secrets.VERCEL_TOKEN }}
```

**Output**
```
Push to main → pipeline starts

ci job:
  ✓ Checkout       2s
  ✓ Setup Node     4s
  ✓ Install        15s
  ✓ Lint           3s   — 0 errors
  ✓ Test           8s   — 32 tests passed
  ✓ Build          18s  — dist/ ready

deploy job (only on main):
  ✓ Deploy to Vercel  12s

→ https://my-react-app.vercel.app  LIVE 🚀
```

---

[View Interview Questions](./interview.md)
