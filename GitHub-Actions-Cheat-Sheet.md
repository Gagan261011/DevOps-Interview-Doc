# GitHub Actions – High Retrieval & Teaching Cheat Sheet

This document is designed for **fast recall and knowledge transfer (KT)**. It assumes you already understand GitHub Actions but need quick retrieval for interviews, teaching, or explaining under pressure. Each section follows a fixed pattern with minimal examples, key fields, and ready-made speaking lines.

---

## Left-Side Topic Map (Overview)

| Topic | Trigger Idea (1 line) | Most Used Keyword / Field |
|-------|----------------------|---------------------------|
| GitHub Actions Architecture | Event-driven automation running workflows in GitHub | `workflow` |
| Workflow File Structure | YAML files in .github/workflows defining CI/CD | `.github/workflows/` |
| Events (on:) | Triggers that start workflows (push, PR, schedule) | `on:` |
| Jobs | Independent units of work running in parallel | `jobs:` |
| Steps | Individual tasks within a job (sequential) | `steps:` |
| Runners (GitHub-hosted vs Self-hosted) | VMs executing workflows (cloud or self-managed) | `runs-on:` |
| Actions (Marketplace) | Reusable community/official automation units | `uses:` |
| Using Actions vs Writing Scripts | Marketplace actions vs custom shell commands | `uses:` vs `run:` |
| Checkout Action | Clone repository code into runner | `actions/checkout@v4` |
| Setup Language (Node/Java/Python) | Install language runtime on runner | `actions/setup-node@v4` |
| Environment Variables | Pass configuration to jobs and steps | `env:` |
| Secrets | Encrypted values for sensitive data | `secrets.TOKEN` |
| Contexts (github, env, steps, needs) | Access workflow metadata and outputs | `${{ github.* }}` |
| Expressions & Conditions (if) | Conditional execution based on expressions | `if:` |
| Matrix Strategy | Run jobs with multiple configurations | `strategy.matrix` |
| Caching | Speed up workflows by caching dependencies | `actions/cache@v4` |
| Artifacts | Share files between jobs or download results | `actions/upload-artifact@v4` |
| Job Dependencies (needs) | Control job execution order | `needs:` |
| Environments & Approvals | Protected deployment targets with manual gates | `environment:` |
| Permissions | Control GitHub token access scope | `permissions:` |
| Reusable Workflows | Share entire workflows across repos | `workflow_call` |
| Composite Actions | Bundle multiple steps into single action | `action.yml` with `runs.using: composite` |
| Manual Triggers (workflow_dispatch) | Manually trigger workflows from UI/API | `workflow_dispatch:` |
| Scheduled Workflows (cron) | Time-based workflow execution | `schedule:` with `cron:` |
| Failure Handling | Control behavior on step/job failure | `continue-on-error`, `if: failure()` |
| Best Practices | Patterns for maintainable, secure workflows | Secrets, caching, matrices |

---

## GitHub Actions Architecture

🔑 **Primary Trigger:** "Event-driven automation platform built into GitHub for CI/CD workflows."

### 1. What it is (Simple Definition)

- GitHub Actions is GitHub's native CI/CD and automation platform
- Workflows respond to repository events (push, PR, issues, etc.)
- Runs on GitHub-hosted or self-hosted runners

### 2. Why / When to Use

- Automate build, test, and deployment pipelines
- Integrate CI/CD directly with GitHub repositories
- No external CI tool setup needed
- Leverage GitHub's native integration (permissions, secrets, environments)

### 3. Key Keywords / Fields

- **Workflow**: YAML file defining automation
- **Event**: Trigger starting workflow
- **Job**: Unit of work with multiple steps
- **Step**: Individual action or command
- **Runner**: VM executing workflow
- **Action**: Reusable automation unit

### 4. Minimal Workflow Example

```yaml
name: Simple CI Workflow
on: [push]
jobs:
  hello:
    runs-on: ubuntu-latest
    steps:
      - name: Say hello
        run: echo "Hello from GitHub Actions!"
```

### 5. Example Scenario

- Developer pushes code to main branch
- Push event triggers workflow automatically
- GitHub spins up Ubuntu runner
- Executes steps (build, test, deploy)
- Results visible in GitHub Actions tab

### 6. Common Interview / KT Line

"GitHub Actions provides event-driven automation directly in GitHub, eliminating the need for external CI/CD tools."

### 7. Closing Line Trigger

"In short, GitHub Actions automates development workflows using GitHub's native event system."

---

## Workflow File Structure

🔑 **Primary Trigger:** "YAML files in .github/workflows/ define what happens when events occur."

### 1. What it is (Simple Definition)

- Workflows are YAML files stored in `.github/workflows/` directory
- Each file defines one workflow with events, jobs, and steps
- File naming is flexible (e.g., `ci.yml`, `deploy.yml`)

### 2. Why / When to Use

- Define all automation in version-controlled YAML
- Separate workflows for different purposes (CI, CD, release)
- Multiple workflows can respond to same events
- Infrastructure as code for CI/CD

### 3. Key Keywords / Fields

- `name:` (workflow name)
- `on:` (triggering events)
- `jobs:` (job definitions)
- `env:` (workflow-level variables)
- `defaults:` (default settings)

### 4. Minimal Workflow Example

```yaml
# .github/workflows/ci.yml
name: CI Pipeline
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '18'

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run tests
        run: npm test
```

### 5. Example Scenario

- Create `.github/workflows/ci.yml` in repository
- Define push and PR triggers
- Add build and test jobs
- Commit workflow file to repository
- Workflow executes on next push automatically

### 6. Common Interview / KT Line

"Workflow files live in .github/workflows/ and define the entire CI/CD pipeline as version-controlled YAML."

### 7. Closing Line Trigger

"In short, workflow files are the blueprints for all GitHub Actions automation."

---

## Events (on:)

🔑 **Primary Trigger:** "Events trigger workflows – push, pull requests, schedules, or manual runs."

### 1. What it is (Simple Definition)

- Events are activities that trigger workflow execution
- Defined in the `on:` section of workflow file
- Can filter events by branches, paths, or types

### 2. Why / When to Use

- Run CI on code pushes or pull requests
- Automate releases on tags
- Schedule periodic tasks (backups, reports)
- Respond to issue creation or comments

### 3. Key Keywords / Fields

- `push:` (code pushed to repo)
- `pull_request:` (PR opened/updated)
- `workflow_dispatch:` (manual trigger)
- `schedule:` (time-based)
- `release:` (release created)
- `issues:` (issue events)
- Filters: `branches:`, `tags:`, `paths:`

### 4. Minimal Workflow Example

```yaml
name: Multi-Event Workflow
on:
  push:
    branches:
      - main
      - 'release/**'
    paths:
      - 'src/**'
      - 'tests/**'
  pull_request:
    types: [opened, synchronize]
  workflow_dispatch:
  schedule:
    - cron: '0 2 * * *'  # Daily at 2 AM UTC

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: echo "Triggered by ${{ github.event_name }}"
```

### 5. Example Scenario

- CI workflow triggers on push to main or develop
- Only runs if changes in `src/` or `tests/` directories
- Also runs on pull requests for code review
- Can be manually triggered for testing
- Scheduled run performs nightly builds

### 6. Common Interview / KT Line

"Events define when workflows run – from code pushes and PRs to schedules and manual triggers with optional filtering."

### 7. Closing Line Trigger

"In short, events are the what, when, and where of workflow execution."

---

## Jobs

🔑 **Primary Trigger:** "Jobs are independent work units that run in parallel by default."

### 1. What it is (Simple Definition)

- Jobs are top-level units of work in a workflow
- Each job runs on its own runner
- By default, jobs run in parallel
- Can define dependencies with `needs:`

### 2. Why / When to Use

- Separate concerns (build, test, deploy)
- Run tasks in parallel for speed
- Isolate environments for different tasks
- Chain jobs with dependencies

### 3. Key Keywords / Fields

- `jobs:` (job definitions section)
- `runs-on:` (runner type)
- `needs:` (job dependencies)
- `steps:` (task list)
- `if:` (conditional execution)
- `strategy:` (matrix builds)

### 4. Minimal Workflow Example

```yaml
name: Multi-Job Workflow
on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build application
        run: npm run build

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run tests
        run: npm test

  deploy:
    runs-on: ubuntu-latest
    needs: [build, test]  # Wait for build and test
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Deploy to production
        run: echo "Deploying..."
```

### 5. Example Scenario

- Workflow has three jobs: build, test, deploy
- Build and test run in parallel (faster)
- Deploy waits for both (uses `needs:`)
- Deploy only runs on main branch (uses `if:`)
- Total time reduced through parallelization

### 6. Common Interview / KT Line

"Jobs run independently in parallel by default, but you can chain them with needs to create deployment pipelines."

### 7. Closing Line Trigger

"In short, jobs organize workflows into parallel or sequential execution units."

---

## Steps

🔑 **Primary Trigger:** "Steps are sequential tasks within a job – actions or shell commands."

### 1. What it is (Simple Definition)

- Steps are individual tasks executed in sequence within a job
- Each step can use an action (`uses:`) or run commands (`run:`)
- Share the same runner and filesystem

### 2. Why / When to Use

- Define exact sequence of operations
- Combine actions and custom scripts
- Build complex workflows step by step
- Share data between steps via filesystem or outputs

### 3. Key Keywords / Fields

- `steps:` (list of tasks)
- `name:` (step description)
- `uses:` (marketplace action)
- `run:` (shell command)
- `with:` (action inputs)
- `env:` (step-level variables)
- `if:` (conditional execution)
- `id:` (reference step in outputs)

### 4. Minimal Workflow Example

```yaml
name: Step Examples
on: [push]

jobs:
  demo:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'

      - name: Install dependencies
        run: npm install

      - name: Run tests
        run: npm test

      - name: Build application
        run: npm run build
        env:
          NODE_ENV: production

      - name: Display build info
        run: |
          echo "Build completed"
          ls -la dist/
```

### 5. Example Scenario

- Job needs to build Node.js application
- Step 1: Checkout code from repository
- Step 2: Install Node.js runtime
- Step 3: Install npm dependencies
- Step 4: Run tests (sequential after deps)
- Step 5: Build production bundle

### 6. Common Interview / KT Line

"Steps define the exact sequence of operations within a job, mixing marketplace actions with custom shell commands."

### 7. Closing Line Trigger

"In short, steps are the building blocks that define what each job actually does."

---

## Runners (GitHub-hosted vs Self-hosted)

🔑 **Primary Trigger:** "Runners are VMs executing workflows – GitHub-hosted or self-hosted."

### 1. What it is (Simple Definition)

- Runners are virtual machines that execute workflow jobs
- GitHub-hosted: Managed by GitHub (Ubuntu, Windows, macOS)
- Self-hosted: Your own infrastructure for custom requirements

### 2. Why / When to Use

- GitHub-hosted: Quick setup, no maintenance, free tier available
- Self-hosted: Custom hardware, proprietary software, network access
- Choose based on cost, performance, and security needs
- Self-hosted for private networks or specific configurations

### 3. Key Keywords / Fields

- `runs-on:`
- `ubuntu-latest`, `windows-latest`, `macos-latest`
- `self-hosted`
- Runner labels for self-hosted

### 4. Minimal Workflow Example

```yaml
name: Runner Examples
on: [push]

jobs:
  # GitHub-hosted runner
  build-linux:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: echo "Running on GitHub-hosted Ubuntu"

  # GitHub-hosted Windows
  build-windows:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4
      - run: echo "Running on GitHub-hosted Windows"

  # Self-hosted runner
  deploy:
    runs-on: [self-hosted, linux, production]
    steps:
      - uses: actions/checkout@v4
      - run: echo "Running on self-hosted runner"
```

### 5. Example Scenario

- Public project uses `ubuntu-latest` for free CI
- Windows-specific builds use `windows-latest`
- Deployment to private network requires self-hosted runner
- Self-hosted runner configured with necessary credentials
- Cost savings for high-volume builds

### 6. Common Interview / KT Line

"GitHub-hosted runners are convenient and free for public repos, while self-hosted runners give control for custom environments."

### 7. Closing Line Trigger

"In short, choose GitHub-hosted for simplicity or self-hosted for control and customization."

---

## Actions (Marketplace)

🔑 **Primary Trigger:** "Actions are reusable automation units from GitHub Marketplace or custom repositories."

### 1. What it is (Simple Definition)

- Actions are packaged automation steps you can reuse
- Available in GitHub Marketplace or create custom
- Simplify workflows by abstracting common tasks
- Versioned with semantic tags or commit SHAs

### 2. Why / When to Use

- Avoid reinventing common tasks (checkout, setup, deploy)
- Leverage community-maintained actions
- Ensure consistency across workflows
- Abstract complexity into reusable components

### 3. Key Keywords / Fields

- `uses:` (action reference)
- `with:` (action inputs)
- `@v4` (version/tag)
- `owner/repo@ref` (action location)

### 4. Minimal Workflow Example

```yaml
name: Using Marketplace Actions
on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      # Official GitHub action
      - name: Checkout code
        uses: actions/checkout@v4

      # Setup action with inputs
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'

      # Community action
      - name: Run Super-Linter
        uses: github/super-linter@v5
        env:
          DEFAULT_BRANCH: main
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      # Action with specific version
      - name: Deploy to AWS
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_KEY }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET }}
          aws-region: us-east-1
```

### 5. Example Scenario

- Need to deploy Node.js app to AWS
- Use `actions/checkout` to get code
- Use `actions/setup-node` for runtime
- Use `aws-actions/configure-aws-credentials` for deployment
- No custom scripts needed – all marketplace actions

### 6. Common Interview / KT Line

"Marketplace actions are like npm packages for CI/CD – reusable, versioned components handling common automation tasks."

### 7. Closing Line Trigger

"In short, actions eliminate boilerplate by providing battle-tested, reusable automation units."

---

## Using Actions vs Writing Scripts

🔑 **Primary Trigger:** "Use marketplace actions for common tasks, write scripts for custom logic."

### 1. What it is (Simple Definition)

- `uses:` references marketplace or custom actions
- `run:` executes shell commands or scripts
- Actions provide abstraction, scripts provide flexibility
- Best practice: combine both as needed

### 2. Why / When to Use

- Actions: Standard tasks (checkout, setup, deploy)
- Scripts: Custom business logic, specific commands
- Actions: Better portability and maintenance
- Scripts: Quick one-offs or proprietary tools

### 3. Key Keywords / Fields

- `uses:` (action reference)
- `run:` (shell command)
- `shell:` (bash, pwsh, python, etc.)
- `working-directory:`

### 4. Minimal Workflow Example

```yaml
name: Actions vs Scripts
on: [push]

jobs:
  demo:
    runs-on: ubuntu-latest
    steps:
      # Using action (preferred for standard tasks)
      - name: Checkout code
        uses: actions/checkout@v4

      # Using action with configuration
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      # Script for custom logic
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      # Script with working directory
      - name: Run custom build script
        run: ./scripts/build.sh
        working-directory: ./app

      # Multi-line script
      - name: Custom processing
        run: |
          echo "Processing files..."
          for file in *.txt; do
            echo "Processing $file"
          done
        shell: bash

      # Using action for deployment
      - name: Deploy to server
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.SSH_KEY }}
          script: |
            cd /var/www
            git pull
            systemctl restart app
```

### 5. Example Scenario

- Use `actions/checkout` instead of scripting git clone
- Use `actions/setup-node` instead of manual Node install
- Write custom script for proprietary build process
- Use deployment action for complex deployment logic
- Balance maintainability (actions) with flexibility (scripts)

### 6. Common Interview / KT Line

"Leverage marketplace actions for standard tasks and write custom scripts only when specific business logic is needed."

### 7. Closing Line Trigger

"In short, actions handle common patterns while scripts handle unique requirements."

---

## Checkout Action

🔑 **Primary Trigger:** "Checkout action clones repository code into the runner workspace."

### 1. What it is (Simple Definition)

- `actions/checkout` is the most used GitHub action
- Clones repository code to runner's workspace
- Required for nearly all workflows that work with code
- Supports submodules, LFS, and custom fetch depth

### 2. Why / When to Use

- First step in most workflows to access code
- Build, test, or deploy operations need source code
- Configure depth for faster checkouts
- Access code from specific branches or commits

### 3. Key Keywords / Fields

- `actions/checkout@v4`
- `with.ref:` (branch/tag/commit)
- `with.fetch-depth:` (history depth)
- `with.submodules:` (include submodules)
- `with.token:` (custom PAT)

### 4. Minimal Workflow Example

```yaml
name: Checkout Examples
on: [push]

jobs:
  basic:
    runs-on: ubuntu-latest
    steps:
      # Basic checkout
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Show files
        run: ls -la

  advanced:
    runs-on: ubuntu-latest
    steps:
      # Checkout with full history
      - name: Checkout with history
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # All history

      # Checkout specific branch
      - name: Checkout develop branch
        uses: actions/checkout@v4
        with:
          ref: develop

      # Checkout with submodules
      - name: Checkout with submodules
        uses: actions/checkout@v4
        with:
          submodules: recursive

      # Checkout different repo
      - name: Checkout another repo
        uses: actions/checkout@v4
        with:
          repository: owner/other-repo
          token: ${{ secrets.PAT }}
```

### 5. Example Scenario

- Workflow needs to build application
- First step: checkout code with `actions/checkout@v4`
- Code appears in runner's workspace directory
- Subsequent steps can access all repository files
- Without checkout, runner has empty workspace

### 6. Common Interview / KT Line

"Checkout is typically the first step in any workflow because it brings your repository code into the runner's workspace."

### 7. Closing Line Trigger

"In short, checkout is the entry point for code-based workflows in GitHub Actions."

---

## Setup Language (Node/Java/Python)

🔑 **Primary Trigger:** "Setup actions install language runtimes and configure environments."

### 1. What it is (Simple Definition)

- Setup actions install specific language versions on runners
- Common: `setup-node`, `setup-python`, `setup-java`, `setup-go`
- Configure PATH, cache dependencies, and version managers
- Ensures consistent runtime across workflow runs

### 2. Why / When to Use

- Install specific Node.js, Python, or Java versions
- Enable dependency caching for faster builds
- Support matrix builds with multiple language versions
- Standardize environment across team

### 3. Key Keywords / Fields

- `actions/setup-node@v4`
- `actions/setup-python@v5`
- `actions/setup-java@v4`
- `with.node-version:`, `with.python-version:`, `with.java-version:`
- `with.cache:` (npm, pip, maven, gradle)

### 4. Minimal Workflow Example

```yaml
name: Multi-Language Setup
on: [push]

jobs:
  node:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
      - run: npm ci
      - run: npm test

  python:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
          cache: 'pip'
      - run: pip install -r requirements.txt
      - run: pytest

  java:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'
          cache: 'maven'
      - run: mvn clean install
```

### 5. Example Scenario

- Node.js application requires version 18
- Add `actions/setup-node@v4` with `node-version: '18'`
- Enable npm caching for faster dependency installation
- Runner now has Node.js 18 and npm available
- Subsequent steps can run npm commands

### 6. Common Interview / KT Line

"Setup actions install language runtimes and enable caching, ensuring consistent environments and faster builds."

### 7. Closing Line Trigger

"In short, setup actions configure the exact language environment your code needs to build and test."

---

## Environment Variables

🔑 **Primary Trigger:** "Environment variables pass configuration to jobs and steps."

### 1. What it is (Simple Definition)

- Environment variables store configuration values
- Defined at workflow, job, or step level
- Accessed via `$VARIABLE_NAME` or `${{ env.VARIABLE_NAME }}`
- Scope: step-level overrides job-level overrides workflow-level

### 2. Why / When to Use

- Configure build environment (NODE_ENV, API endpoints)
- Avoid hardcoding values in scripts
- Share configuration across multiple steps
- Enable environment-specific builds

### 3. Key Keywords / Fields

- `env:` (define variables)
- `${{ env.VAR_NAME }}` (access in workflow)
- `$VAR_NAME` (access in scripts)
- `GITHUB_ENV` (set for subsequent steps)

### 4. Minimal Workflow Example

```yaml
name: Environment Variables
on: [push]

# Workflow-level env vars
env:
  GLOBAL_VAR: 'workflow-value'
  NODE_VERSION: '18'

jobs:
  demo:
    runs-on: ubuntu-latest
    # Job-level env vars
    env:
      JOB_VAR: 'job-value'
      API_URL: 'https://api.example.com'

    steps:
      - uses: actions/checkout@v4

      # Step-level env var
      - name: Build with environment
        env:
          STEP_VAR: 'step-value'
          NODE_ENV: 'production'
        run: |
          echo "Global: $GLOBAL_VAR"
          echo "Job: $JOB_VAR"
          echo "Step: $STEP_VAR"
          echo "Building for $NODE_ENV"

      # Access in expression
      - name: Use in expression
        run: echo "API URL is ${{ env.API_URL }}"

      # Set env var for subsequent steps
      - name: Set dynamic env var
        run: echo "BUILD_NUMBER=12345" >> $GITHUB_ENV

      - name: Use dynamic env var
        run: echo "Build number is $BUILD_NUMBER"
```

### 5. Example Scenario

- Application needs different API URLs per environment
- Define `API_URL` in env vars
- Dev workflow: `API_URL=https://dev-api.com`
- Prod workflow: `API_URL=https://prod-api.com`
- Code reads from environment without hardcoding

### 6. Common Interview / KT Line

"Environment variables centralize configuration and enable the same workflow to work across different environments."

### 7. Closing Line Trigger

"In short, env vars provide flexible configuration without hardcoding values in scripts."

---

## Secrets

🔑 **Primary Trigger:** "Secrets store encrypted sensitive values like passwords and tokens."

### 1. What it is (Simple Definition)

- Secrets are encrypted environment variables
- Configured in repository or organization settings
- Never exposed in logs or accessible to forks
- Accessed via `${{ secrets.SECRET_NAME }}`

### 2. Why / When to Use

- Store API keys, passwords, tokens securely
- Deploy to cloud providers with credentials
- Access private resources during builds
- Comply with security policies

### 3. Key Keywords / Fields

- `${{ secrets.SECRET_NAME }}`
- `secrets.GITHUB_TOKEN` (automatic)
- Repository settings → Secrets and variables
- Organization-level secrets
- Environment-specific secrets

### 4. Minimal Workflow Example

```yaml
name: Using Secrets
on: [push]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Using secret in env var
      - name: Configure AWS credentials
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        run: |
          aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
          aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY

      # Using secret in action input
      - name: Deploy to production
        uses: some-deployment-action@v1
        with:
          api-token: ${{ secrets.DEPLOY_TOKEN }}
          environment: production

      # Using default GITHUB_TOKEN
      - name: Create release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          gh release create v1.0.0 --notes "Release notes"

      # Multiple secrets
      - name: Configure application
        env:
          DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
          API_KEY: ${{ secrets.API_KEY }}
          SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}
        run: ./deploy.sh
```

### 5. Example Scenario

- Workflow deploys to AWS requiring credentials
- Add `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` as repository secrets
- Reference in workflow: `${{ secrets.AWS_ACCESS_KEY_ID }}`
- Values encrypted and never shown in logs
- Deployment succeeds securely

### 6. Common Interview / KT Line

"Secrets encrypt sensitive values and GitHub ensures they never appear in logs, making workflows secure by default."

### 7. Closing Line Trigger

"In short, secrets enable secure automation without exposing credentials in code or logs."

---

## Contexts (github, env, steps, needs)

🔑 **Primary Trigger:** "Contexts provide access to workflow metadata, variables, and outputs."

### 1. What it is (Simple Definition)

- Contexts are objects containing workflow information
- Accessed via `${{ context.property }}` syntax
- Common contexts: `github`, `env`, `steps`, `needs`, `job`, `runner`
- Enable dynamic behavior based on runtime information

### 2. Why / When to Use

- Access event data (branch, commit, PR info)
- Reference outputs from previous steps or jobs
- Make decisions based on workflow context
- Build dynamic workflows responding to conditions

### 3. Key Keywords / Fields

- `${{ github.* }}` (event and repo info)
- `${{ env.* }}` (environment variables)
- `${{ steps.<id>.outputs.* }}` (step outputs)
- `${{ needs.<job>.outputs.* }}` (job outputs)
- `${{ runner.* }}` (runner information)
- `${{ job.* }}` (current job info)

### 4. Minimal Workflow Example

```yaml
name: Context Examples
on: [push, pull_request]

jobs:
  context-demo:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.version.outputs.value }}
    steps:
      - uses: actions/checkout@v4

      # GitHub context
      - name: Show GitHub context
        run: |
          echo "Event: ${{ github.event_name }}"
          echo "Branch: ${{ github.ref_name }}"
          echo "Commit: ${{ github.sha }}"
          echo "Actor: ${{ github.actor }}"
          echo "Repo: ${{ github.repository }}"

      # Set and use step output
      - name: Generate version
        id: version
        run: echo "value=1.2.3" >> $GITHUB_OUTPUT

      - name: Use step output
        run: echo "Version is ${{ steps.version.outputs.value }}"

      # Runner context
      - name: Show runner info
        run: |
          echo "OS: ${{ runner.os }}"
          echo "Arch: ${{ runner.arch }}"
          echo "Temp: ${{ runner.temp }}"

  use-output:
    runs-on: ubuntu-latest
    needs: context-demo
    steps:
      # Access output from previous job
      - name: Use job output
        run: echo "Version from previous job: ${{ needs.context-demo.outputs.version }}"
```

### 5. Example Scenario

- Need to conditionally deploy based on branch
- Use `github.ref_name` context to check branch
- Step outputs version number for next job
- Next job references output via `needs` context
- Workflow adapts behavior based on runtime context

### 6. Common Interview / KT Line

"Contexts provide runtime information about the workflow, enabling dynamic decisions and data flow between steps and jobs."

### 7. Closing Line Trigger

"In short, contexts are GitHub Actions' way of exposing workflow metadata and enabling communication."

---

## Expressions & Conditions (if)

🔑 **Primary Trigger:** "Conditional execution using if with expressions and functions."

### 1. What it is (Simple Definition)

- `if:` conditionally executes jobs or steps
- Expressions use `${{ }}` syntax with operators and functions
- Boolean logic: `&&`, `||`, `!`
- Built-in functions: `contains()`, `startsWith()`, `success()`, `failure()`

### 2. Why / When to Use

- Skip steps based on conditions (branch, event type)
- Run deployment only on main branch
- Execute cleanup on failure
- Conditional logic without custom scripts

### 3. Key Keywords / Fields

- `if:` (condition)
- `${{ expression }}`
- `success()`, `failure()`, `cancelled()`, `always()`
- `contains()`, `startsWith()`, `endsWith()`
- `fromJSON()`, `toJSON()`

### 4. Minimal Workflow Example

```yaml
name: Conditional Execution
on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run tests
        run: npm test

      # Run only on main branch
      - name: Deploy to production
        if: github.ref == 'refs/heads/main'
        run: echo "Deploying to production"

      # Run only on pull requests
      - name: Comment on PR
        if: github.event_name == 'pull_request'
        run: echo "This is a PR"

      # Multiple conditions
      - name: Deploy staging
        if: github.ref == 'refs/heads/develop' && success()
        run: echo "Deploying to staging"

      # Using contains function
      - name: Skip for WIP
        if: "!contains(github.event.head_commit.message, '[skip ci]')"
        run: echo "Running build"

  cleanup:
    runs-on: ubuntu-latest
    needs: build
    if: failure()  # Run only if build fails
    steps:
      - name: Notify failure
        run: echo "Build failed, sending notification"

  always-run:
    runs-on: ubuntu-latest
    needs: build
    if: always()  # Always run
    steps:
      - name: Cleanup resources
        run: echo "Cleaning up"
```

### 5. Example Scenario

- Workflow runs on all branches
- Build and test run everywhere
- Deployment step only on main branch: `if: github.ref == 'refs/heads/main'`
- Notification step only on failure: `if: failure()`
- Conditional logic without complex scripts

### 6. Common Interview / KT Line

"The if condition enables surgical control over workflow execution, running steps only when specific conditions are met."

### 7. Closing Line Trigger

"In short, expressions and conditions make workflows intelligent and context-aware."

---

## Matrix Strategy

🔑 **Primary Trigger:** "Matrix strategy runs jobs with multiple configurations in parallel."

### 1. What it is (Simple Definition)

- Matrix creates multiple job variations from configuration arrays
- Test across multiple OS, language versions, or custom dimensions
- Runs in parallel for efficiency
- Can exclude specific combinations

### 2. Why / When to Use

- Test on multiple operating systems (Linux, Windows, macOS)
- Test multiple language or framework versions
- Validate cross-platform compatibility
- Parallel execution saves time

### 3. Key Keywords / Fields

- `strategy.matrix:`
- `strategy.matrix.exclude:`
- `strategy.matrix.include:`
- `strategy.fail-fast:` (stop on first failure)
- `strategy.max-parallel:`
- `${{ matrix.variable }}`

### 4. Minimal Workflow Example

```yaml
name: Matrix Strategy
on: [push]

jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node-version: [16, 18, 20]
        exclude:
          - os: windows-latest
            node-version: 16
        include:
          - os: ubuntu-latest
            node-version: 20
            experimental: true

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Show configuration
        run: |
          echo "OS: ${{ matrix.os }}"
          echo "Node: ${{ matrix.node-version }}"
```

### 5. Example Scenario

- Library needs testing on Node 16, 18, 20
- Also needs testing on Linux, Windows, macOS
- Matrix creates 9 job combinations (3 OS × 3 versions)
- All run in parallel
- Results show compatibility across platforms
- Can exclude Windows + Node 16 if not needed

### 6. Common Interview / KT Line

"Matrix strategy multiplies jobs across configurations, enabling comprehensive testing without writing repetitive workflows."

### 7. Closing Line Trigger

"In short, matrices provide efficient, parallel testing across multiple dimensions."

---

## Caching

🔑 **Primary Trigger:** "Cache dependencies between workflow runs for faster builds."

### 1. What it is (Simple Definition)

- `actions/cache` stores and restores files between workflow runs
- Typically used for dependencies (npm, pip, maven)
- Significantly speeds up builds by avoiding re-downloads
- Cache keys ensure proper invalidation

### 2. Why / When to Use

- Speed up dependency installation
- Reduce network traffic and build times
- Cache node_modules, .m2, pip packages
- Automatic caching available in setup actions

### 3. Key Keywords / Fields

- `actions/cache@v4`
- `with.path:` (directories to cache)
- `with.key:` (cache key)
- `with.restore-keys:` (fallback keys)
- Built-in caching in setup actions (`cache: 'npm'`)

### 4. Minimal Workflow Example

```yaml
name: Caching Examples
on: [push]

jobs:
  # Manual caching
  manual-cache:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Cache node modules
        uses: actions/cache@v4
        with:
          path: ~/.npm
          key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-node-

      - name: Install dependencies
        run: npm ci

  # Built-in caching with setup actions
  auto-cache:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node with cache
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'

      - run: npm ci

  # Multiple cache paths
  multi-cache:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Cache multiple directories
        uses: actions/cache@v4
        with:
          path: |
            ~/.npm
            ~/.cache/pip
            vendor/bundle
          key: ${{ runner.os }}-multi-${{ hashFiles('**/lockfile') }}

      - run: npm ci
```

### 5. Example Scenario

- Node.js project with many dependencies
- First run: npm install takes 2 minutes
- Enable caching with cache key based on package-lock.json
- Second run: cache hit, dependencies restored in 10 seconds
- Build time reduced from 2 minutes to 10 seconds
- Cache invalidates automatically when package-lock.json changes

### 6. Common Interview / KT Line

"Caching dramatically speeds up workflows by storing dependencies between runs, with automatic invalidation when lockfiles change."

### 7. Closing Line Trigger

"In short, caching is essential for fast, efficient CI/CD pipelines."

---

## Artifacts

🔑 **Primary Trigger:** "Artifacts share files between jobs or persist build outputs."

### 1. What it is (Simple Definition)

- Artifacts are files persisted after workflow completion
- Shared between jobs in same workflow
- Downloaded from GitHub UI or API
- Retention period configurable (default 90 days)

### 2. Why / When to Use

- Share build outputs between jobs (build → deploy)
- Store test results, logs, coverage reports
- Download compiled binaries
- Debug failed workflows

### 3. Key Keywords / Fields

- `actions/upload-artifact@v4`
- `actions/download-artifact@v4`
- `with.name:` (artifact name)
- `with.path:` (files to upload)
- `with.retention-days:` (storage period)

### 4. Minimal Workflow Example

```yaml
name: Artifacts Example
on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build application
        run: |
          mkdir -p dist
          echo "Build output" > dist/app.js
          npm run build

      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build-output
          path: dist/
          retention-days: 30

      - name: Upload test results
        uses: actions/upload-artifact@v4
        with:
          name: test-results
          path: |
            coverage/
            test-report.xml

  deploy:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Download build artifacts
        uses: actions/download-artifact@v4
        with:
          name: build-output
          path: ./dist

      - name: Deploy to server
        run: |
          ls -la dist/
          echo "Deploying artifacts..."
```

### 5. Example Scenario

- Build job compiles application to dist/ folder
- Upload dist/ as artifact named "build-output"
- Deploy job downloads "build-output" artifact
- Deploy job deploys files without rebuilding
- Build artifacts also downloadable from GitHub UI

### 6. Common Interview / KT Line

"Artifacts persist files between jobs and make build outputs downloadable, enabling job separation and debugging."

### 7. Closing Line Trigger

"In short, artifacts are GitHub Actions' way of passing files through the pipeline."

---

## Job Dependencies (needs)

🔑 **Primary Trigger:** "Control job execution order with needs for sequential workflows."

### 1. What it is (Simple Definition)

- `needs:` creates dependencies between jobs
- Dependent jobs wait for prerequisites to complete
- By default, jobs run in parallel
- Enables sequential or mixed parallel/sequential workflows

### 2. Why / When to Use

- Create deployment pipelines (build → test → deploy)
- Ensure jobs run in specific order
- Pass data between jobs via outputs
- Fail fast if prerequisite fails

### 3. Key Keywords / Fields

- `needs:` (single job or array)
- `needs: [job1, job2]` (multiple dependencies)
- `outputs:` (expose data to dependent jobs)
- `needs.<job>.outputs.<name>` (access outputs)

### 4. Minimal Workflow Example

```yaml
name: Job Dependencies
on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.version.outputs.number }}
    steps:
      - uses: actions/checkout@v4
      - name: Build
        run: npm run build
      - name: Set version
        id: version
        run: echo "number=1.2.3" >> $GITHUB_OUTPUT

  test:
    runs-on: ubuntu-latest
    needs: build  # Wait for build
    steps:
      - uses: actions/checkout@v4
      - run: npm test

  deploy-staging:
    runs-on: ubuntu-latest
    needs: [build, test]  # Wait for both
    steps:
      - name: Deploy to staging
        run: |
          echo "Deploying version ${{ needs.build.outputs.version }}"
          echo "Deploying to staging..."

  deploy-production:
    runs-on: ubuntu-latest
    needs: deploy-staging  # Wait for staging
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Deploy to production
        run: echo "Deploying to production..."
```

### 5. Example Scenario

- Workflow has build, test, and deploy jobs
- Test needs successful build
- Staging deploy needs both build and test
- Production deploy needs staging
- If build fails, all downstream jobs skip
- Total workflow time optimized with parallelism where possible

### 6. Common Interview / KT Line

"The needs keyword chains jobs into pipelines, ensuring proper execution order while allowing parallelism where possible."

### 7. Closing Line Trigger

"In short, needs transforms parallel jobs into coordinated deployment pipelines."

---

## Environments & Approvals

🔑 **Primary Trigger:** "Environments add protection rules and manual approval gates."

### 1. What it is (Simple Definition)

- Environments are deployment targets with protection rules
- Configure in repository settings → Environments
- Support manual approvals, wait timers, and branch restrictions
- Track deployments in GitHub UI

### 2. Why / When to Use

- Require manual approval before production deployment
- Restrict deployments to specific branches
- Add environment-specific secrets
- Audit deployment history

### 3. Key Keywords / Fields

- `environment:` (environment name)
- `environment.name:`
- `environment.url:` (deployment URL)
- Repository settings for protection rules
- Required reviewers
- Wait timer
- Deployment branches

### 4. Minimal Workflow Example

```yaml
name: Deployment with Environments
on: [push]

jobs:
  deploy-staging:
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.example.com
    steps:
      - uses: actions/checkout@v4
      - name: Deploy to staging
        run: echo "Deploying to staging..."

  deploy-production:
    runs-on: ubuntu-latest
    needs: deploy-staging
    if: github.ref == 'refs/heads/main'
    environment:
      name: production
      url: https://example.com
    steps:
      - name: Deploy to production
        env:
          PROD_API_KEY: ${{ secrets.PROD_API_KEY }}
        run: |
          echo "Deploying to production..."
          echo "Using production secrets"
```

**Environment Configuration (in GitHub UI):**
- Repository Settings → Environments → New environment
- Name: "production"
- Protection rules:
  - Required reviewers: 2 approvers
  - Wait timer: 5 minutes
  - Deployment branches: only main
- Environment secrets: PROD_API_KEY

### 5. Example Scenario

- Production environment requires manager approval
- Workflow reaches production job
- GitHub pauses and requests approval
- Manager reviews and approves
- Job continues with deployment
- Deployment tracked in environment history

### 6. Common Interview / KT Line

"Environments add protection layers to deployments with manual approvals, branch restrictions, and environment-specific secrets."

### 7. Closing Line Trigger

"In short, environments provide controlled, auditable deployment workflows with human gates."

---

## Permissions

🔑 **Primary Trigger:** "Permissions control what the GITHUB_TOKEN can access."

### 1. What it is (Simple Definition)

- `permissions:` controls GITHUB_TOKEN scope
- Limits what workflows can access (issues, PRs, packages, etc.)
- Follows principle of least privilege
- Can be set at workflow or job level

### 2. Why / When to Use

- Restrict token access for security
- Grant only necessary permissions
- Prevent accidental or malicious modifications
- Required for certain actions (create releases, write packages)

### 3. Key Keywords / Fields

- `permissions:`
- `contents:` (read/write repository)
- `issues:` (create/modify issues)
- `pull-requests:` (create/modify PRs)
- `packages:` (publish packages)
- `deployments:` (create deployments)
- `read-all` / `write-all` / `none`

### 4. Minimal Workflow Example

```yaml
name: Permissions Example
on: [push]

# Workflow-level permissions (default for all jobs)
permissions:
  contents: read
  pull-requests: write

jobs:
  # Job uses workflow permissions
  comment-pr:
    runs-on: ubuntu-latest
    steps:
      - name: Comment on PR
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          gh pr comment ${{ github.event.pull_request.number }} \
            --body "Build completed successfully"

  # Job overrides permissions
  publish:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      packages: write
    steps:
      - uses: actions/checkout@v4

      - name: Create release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: gh release create v1.0.0

      - name: Publish package
        run: npm publish

  # Job with minimal permissions
  test:
    runs-on: ubuntu-latest
    permissions:
      contents: read  # Read-only
    steps:
      - uses: actions/checkout@v4
      - run: npm test
```

### 5. Example Scenario

- Workflow needs to create release on GitHub
- Default token lacks write permissions
- Add `permissions: contents: write`
- Token can now create releases
- Other jobs remain read-only for security
- Follows least privilege principle

### 6. Common Interview / KT Line

"Permissions explicitly control GITHUB_TOKEN access, ensuring workflows only have necessary privileges for security."

### 7. Closing Line Trigger

"In short, permissions enforce least privilege access for workflow security."

---

## Reusable Workflows

🔑 **Primary Trigger:** "Reusable workflows share entire workflows across repositories."

### 1. What it is (Simple Definition)

- Reusable workflows are called by other workflows
- Defined with `workflow_call` trigger
- Accept inputs and secrets as parameters
- Enable DRY principle across repositories

### 2. Why / When to Use

- Standardize workflows across organization
- Centralize maintenance in single workflow
- Share common patterns (build, test, deploy)
- Reduce duplication

### 3. Key Keywords / Fields

- `on.workflow_call:` (make workflow reusable)
- `inputs:` (workflow parameters)
- `secrets:` (required secrets)
- `uses:` (call reusable workflow)
- `with:` (pass inputs)

### 4. Minimal Workflow Example

**Reusable workflow (.github/workflows/reusable-deploy.yml):**
```yaml
name: Reusable Deployment
on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string
      version:
        required: false
        type: string
        default: 'latest'
    secrets:
      deploy-token:
        required: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}
    steps:
      - uses: actions/checkout@v4

      - name: Deploy version ${{ inputs.version }}
        env:
          TOKEN: ${{ secrets.deploy-token }}
        run: |
          echo "Deploying ${{ inputs.version }} to ${{ inputs.environment }}"
          ./deploy.sh
```

**Caller workflow (.github/workflows/main.yml):**
```yaml
name: Main Pipeline
on: [push]

jobs:
  deploy-staging:
    uses: ./.github/workflows/reusable-deploy.yml
    with:
      environment: staging
      version: '1.2.3'
    secrets:
      deploy-token: ${{ secrets.STAGING_TOKEN }}

  deploy-prod:
    needs: deploy-staging
    uses: ./.github/workflows/reusable-deploy.yml
    with:
      environment: production
    secrets:
      deploy-token: ${{ secrets.PROD_TOKEN }}
```

### 5. Example Scenario

- Organization has 50 repositories with similar deployment
- Create reusable deployment workflow in central repo
- Each repo calls reusable workflow with environment-specific inputs
- Update deployment logic once, affects all repos
- Consistency and maintenance improved

### 6. Common Interview / KT Line

"Reusable workflows eliminate duplication by letting you call shared workflows across repositories with parameterized inputs."

### 7. Closing Line Trigger

"In short, reusable workflows are functions for CI/CD, enabling organizational standardization."

---

## Composite Actions

🔑 **Primary Trigger:** "Composite actions bundle multiple steps into a single reusable action."

### 1. What it is (Simple Definition)

- Composite actions package multiple steps as one action
- Defined in `action.yml` file
- Can be shared via Marketplace or referenced locally
- Accept inputs and produce outputs

### 2. Why / When to Use

- Reuse common step sequences within workflows
- Create organization-specific actions
- Abstract complexity into simple interfaces
- Share across projects or publicly

### 3. Key Keywords / Fields

- `action.yml` (action definition)
- `runs.using: composite`
- `inputs:` (action parameters)
- `outputs:` (action outputs)
- `runs.steps:` (steps to execute)

### 4. Minimal Workflow Example

**Composite action (.github/actions/setup-app/action.yml):**
```yaml
name: Setup Application
description: Setup Node.js and install dependencies

inputs:
  node-version:
    description: Node.js version
    required: false
    default: '18'

outputs:
  cache-hit:
    description: Whether cache was hit
    value: ${{ steps.cache.outputs.cache-hit }}

runs:
  using: composite
  steps:
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ inputs.node-version }}
        cache: 'npm'

    - name: Cache dependencies
      id: cache
      uses: actions/cache@v4
      with:
        path: node_modules
        key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}

    - name: Install dependencies
      if: steps.cache.outputs.cache-hit != 'true'
      shell: bash
      run: npm ci
```

**Using composite action:**
```yaml
name: Use Composite Action
on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Use local composite action
      - name: Setup application
        uses: ./.github/actions/setup-app
        with:
          node-version: '20'

      - name: Build
        run: npm run build

      - name: Test
        run: npm test
```

### 5. Example Scenario

- Multiple workflows need Node.js setup with caching
- Create composite action bundling setup-node, caching, npm ci
- Reference composite action in all workflows
- Update setup logic once, affects all users
- Workflows become cleaner and more maintainable

### 6. Common Interview / KT Line

"Composite actions package repetitive step sequences into reusable units, making workflows cleaner and more maintainable."

### 7. Closing Line Trigger

"In short, composite actions are building blocks that encapsulate common patterns."

---

## Manual Triggers (workflow_dispatch)

🔑 **Primary Trigger:** "Manually trigger workflows from GitHub UI or API with custom inputs."

### 1. What it is (Simple Definition)

- `workflow_dispatch` enables manual workflow triggers
- Appears as "Run workflow" button in GitHub Actions tab
- Supports custom input parameters
- Useful for on-demand tasks

### 2. Why / When to Use

- Run deployments manually
- Execute maintenance tasks on-demand
- Test workflows without pushing code
- Provide self-service automation to team

### 3. Key Keywords / Fields

- `on.workflow_dispatch:`
- `inputs:` (manual input parameters)
- Input types: `string`, `boolean`, `choice`, `environment`
- `github.event.inputs.<name>` (access inputs)

### 4. Minimal Workflow Example

```yaml
name: Manual Deployment
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Deployment environment'
        required: true
        type: choice
        options:
          - development
          - staging
          - production
      version:
        description: 'Version to deploy'
        required: true
        type: string
        default: 'latest'
      dry-run:
        description: 'Perform dry run'
        required: false
        type: boolean
        default: false

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: ${{ github.event.inputs.environment }}
    steps:
      - uses: actions/checkout@v4

      - name: Show inputs
        run: |
          echo "Environment: ${{ github.event.inputs.environment }}"
          echo "Version: ${{ github.event.inputs.version }}"
          echo "Dry run: ${{ github.event.inputs.dry-run }}"

      - name: Deploy
        run: |
          if [ "${{ github.event.inputs.dry-run }}" == "true" ]; then
            echo "DRY RUN: Would deploy ${{ github.event.inputs.version }}"
          else
            echo "Deploying ${{ github.event.inputs.version }} to ${{ github.event.inputs.environment }}"
            ./deploy.sh
          fi
```

### 5. Example Scenario

- Need to deploy specific version to staging
- Go to Actions tab → Manual Deployment
- Click "Run workflow"
- Select environment: staging
- Enter version: v1.2.3
- Click Run
- Workflow executes with specified parameters

### 6. Common Interview / KT Line

"Workflow dispatch provides self-service automation through the GitHub UI, letting users trigger workflows with custom parameters."

### 7. Closing Line Trigger

"In short, manual triggers give teams on-demand control over workflow execution."

---

## Scheduled Workflows (cron)

🔑 **Primary Trigger:** "Schedule workflows with cron syntax for periodic execution."

### 1. What it is (Simple Definition)

- `schedule:` trigger runs workflows on time-based schedule
- Uses standard cron syntax
- Runs on default branch only
- UTC timezone

### 2. Why / When to Use

- Nightly builds or tests
- Periodic cleanup tasks
- Scheduled reports or backups
- Dependency updates
- Health checks

### 3. Key Keywords / Fields

- `on.schedule:`
- `cron:` (cron expression)
- Format: `minute hour day month weekday`
- `* * * * *` (all fields)

### 4. Minimal Workflow Example

```yaml
name: Scheduled Tasks
on:
  schedule:
    # Daily at 2 AM UTC
    - cron: '0 2 * * *'
    # Every Monday at 9 AM UTC
    - cron: '0 9 * * 1'
    # Every 6 hours
    - cron: '0 */6 * * *'
  # Allow manual trigger for testing
  workflow_dispatch:

jobs:
  nightly-build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run nightly build
        run: |
          echo "Running nightly build at $(date)"
          npm run build

      - name: Run extended tests
        run: npm run test:extended

  cleanup:
    runs-on: ubuntu-latest
    steps:
      - name: Cleanup old artifacts
        run: echo "Cleaning up old build artifacts..."

  dependency-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Check for dependency updates
        run: |
          npm outdated
          echo "Dependency check completed"
```

**Common cron patterns:**
```yaml
# Every day at midnight UTC
- cron: '0 0 * * *'

# Every weekday at 9 AM UTC
- cron: '0 9 * * 1-5'

# First day of every month at 2 AM UTC
- cron: '0 2 1 * *'

# Every 15 minutes
- cron: '*/15 * * * *'
```

### 5. Example Scenario

- Need nightly builds for comprehensive testing
- Add schedule: `cron: '0 2 * * *'` (2 AM daily)
- Workflow runs automatically every night
- Extended test suite executes when developers offline
- Issues detected early with nightly feedback
- No manual intervention required

### 6. Common Interview / KT Line

"Scheduled workflows use cron syntax to automate periodic tasks like nightly builds, cleanups, or dependency checks."

### 7. Closing Line Trigger

"In short, schedules enable time-based automation for recurring maintenance and testing tasks."

---

## Failure Handling

🔑 **Primary Trigger:** "Control workflow behavior on failures with continue-on-error and status checks."

### 1. What it is (Simple Definition)

- By default, step failure stops job execution
- `continue-on-error` allows job to continue despite failures
- `if: failure()` runs steps only on failure
- `if: always()` runs steps regardless of status

### 2. Why / When to Use

- Allow non-critical steps to fail
- Run cleanup steps on failure
- Send notifications when builds fail
- Implement retry logic

### 3. Key Keywords / Fields

- `continue-on-error:` (true/false)
- `if: failure()` (run on failure)
- `if: success()` (run on success)
- `if: always()` (always run)
- `if: cancelled()` (run if cancelled)
- `timeout-minutes:`

### 4. Minimal Workflow Example

```yaml
name: Failure Handling
on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v4

      - name: Build application
        run: npm run build

      # Non-critical step - allow failure
      - name: Optional lint check
        run: npm run lint
        continue-on-error: true

      - name: Run tests
        run: npm test

      # Run only on failure
      - name: Upload logs on failure
        if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: failure-logs
          path: logs/

      # Always run cleanup
      - name: Cleanup
        if: always()
        run: rm -rf temp/

  notify:
    runs-on: ubuntu-latest
    needs: build
    if: failure()  # Only run if build fails
    steps:
      - name: Send failure notification
        run: |
          echo "Build failed - sending notification"
          curl -X POST ${{ secrets.SLACK_WEBHOOK }} \
            -d '{"text":"Build failed for ${{ github.repository }}"}'

  # Job that continues despite dependency failure
  cleanup:
    runs-on: ubuntu-latest
    needs: build
    if: always()
    steps:
      - name: Always cleanup resources
        run: echo "Cleaning up regardless of build status"
```

### 5. Example Scenario

- Build workflow runs tests that occasionally flake
- Non-critical linting step added with `continue-on-error: true`
- If lint fails, workflow continues
- If tests fail, upload logs with `if: failure()`
- Send Slack notification on failure
- Cleanup always runs with `if: always()`

### 6. Common Interview / KT Line

"Failure handling lets workflows gracefully handle errors, run cleanup tasks, and notify teams while preventing brittle pipelines."

### 7. Closing Line Trigger

"In short, failure handling provides resilience and observability in workflow execution."

---

## Best Practices

🔑 **Primary Trigger:** "Guidelines for secure, efficient, and maintainable GitHub Actions workflows."

### 1. What it is (Simple Definition)

- Collection of proven patterns and security principles
- Improves reliability, performance, and maintainability
- Reduces costs and security risks
- Industry-standard recommendations

### 2. Why / When to Use

- Start projects with solid foundation
- Ensure team consistency
- Pass security audits
- Optimize workflow performance and costs

### 3. Key Keywords / Fields

**Security:**
- Pin action versions (`@v4` or `@sha`)
- Minimal permissions
- Secret management
- No secrets in logs

**Performance:**
- Caching
- Parallel jobs
- Minimal checkout depth
- Self-hosted runners for heavy loads

**Maintainability:**
- Reusable workflows
- Clear job/step names
- Documentation
- Version control

### 4. Minimal Workflow Example

```yaml
name: Best Practices Example
on:
  push:
    branches: [main]
  pull_request:

# Minimal permissions
permissions:
  contents: read
  pull-requests: write

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      # Pin to specific SHA for security
      - name: Checkout code
        uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11  # v4.1.1

      # Use caching for speed
      - name: Setup Node.js
        uses: actions/setup-node@60edb5dd545a775178f52524783378180af0d1f8  # v4.0.2
        with:
          node-version: '18'
          cache: 'npm'

      # Clear step names
      - name: Install dependencies
        run: npm ci

      # Non-blocking lint
      - name: Run linter
        run: npm run lint
        continue-on-error: true

  test:
    runs-on: ubuntu-latest
    timeout-minutes: 10  # Prevent runaway jobs
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      # Store test results
      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: test-results
          path: test-results/

  # Only deploy from main branch
  deploy:
    runs-on: ubuntu-latest
    needs: [lint, test]
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    environment: production
    permissions:
      contents: read
      deployments: write
    steps:
      - uses: actions/checkout@v4

      - name: Deploy to production
        env:
          DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}
        run: |
          # Never echo secrets
          echo "Deploying to production..."
          ./deploy.sh
```

**Key Best Practices Checklist:**

1. **Security:**
   - Pin actions to specific versions or SHAs
   - Use minimal permissions
   - Never log secrets
   - Use environment protection for production
   - Review third-party actions before use

2. **Performance:**
   - Enable caching (npm, pip, maven)
   - Use matrix for parallel testing
   - Shallow clone with `fetch-depth: 1`
   - Use self-hosted runners for frequent workflows
   - Set appropriate `timeout-minutes`

3. **Reliability:**
   - Use `continue-on-error` for non-critical steps
   - Implement retry logic for flaky steps
   - Add status checks and notifications
   - Use `if: always()` for cleanup

4. **Maintainability:**
   - Name all jobs and steps clearly
   - Use reusable workflows for common patterns
   - Document complex logic with comments
   - Version control workflow files
   - Keep workflows DRY (Don't Repeat Yourself)

5. **Cost Optimization:**
   - Use public runners when possible
   - Cancel redundant runs on new pushes
   - Set job `concurrency` to prevent duplicate runs
   - Clean up old artifacts

### 5. Example Scenario

- Team follows best practices from day one
- Pin actions to SHA for supply chain security
- Enable caching for 3x faster builds
- Use minimal permissions for least privilege
- Set up environment protection for production
- Deploy only from main branch
- Store test results as artifacts
- Workflows reliable, fast, and secure

### 6. Common Interview / KT Line

"Following GitHub Actions best practices means secure, fast, maintainable workflows that save costs and reduce risks."

### 7. Closing Line Trigger

"In short, best practices transform workflows from scripts into professional, production-ready CI/CD pipelines."

---

## Closing Notes

This cheat sheet covers **core GitHub Actions concepts** for fast retrieval, spoken explanation, and knowledge transfer. Each section follows a consistent pattern to help you explain confidently and write workflows fluently.

**How to Use This Document:**

- **Before interviews**: Read primary triggers to activate memory
- **During KT sessions**: Use example workflows verbatim
- **Quick lookups**: Reference key fields and syntax
- **Teaching juniors**: Follow the structure for clear explanations

**Practice Tips:**

- Read each section's primary trigger and try explaining without notes
- Copy example workflows and run them in test repository
- Speak sections aloud to practice verbal fluency
- Combine multiple topics for comprehensive explanations

**Remember:**
- GitHub Actions is **event-driven** (workflows trigger on events)
- Workflows are **YAML files** in `.github/workflows/`
- Jobs run in **parallel** by default (use `needs:` for sequence)
- Use **marketplace actions** for common tasks
- **Cache** dependencies for speed
- **Secrets** for sensitive data
- **Environments** for deployment protection

---

**End of GitHub Actions – High Retrieval & Teaching Cheat Sheet**
