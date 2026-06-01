# GitHub Actions

## Table of Contents

* Introduction
* What is GitHub Actions?
* Why Do We Need GitHub Actions?
* Key Concepts
* Workflow Architecture
* Components of GitHub Actions
* Workflow Lifecycle
* Common Use Cases
* Advantages
* Disadvantages
* Best Practices
* Interview Questions
* Conclusion

---

# Introduction

GitHub Actions is GitHub's built-in **Continuous Integration (CI)** and **Continuous Deployment (CD)** platform.

It allows developers to automate software development workflows directly from a GitHub repository.

With GitHub Actions, tasks such as building code, running tests, generating reports, packaging applications, and deploying software can be executed automatically whenever specific events occur.

---

# What is GitHub Actions?

GitHub Actions is an automation framework that enables developers to create workflows triggered by GitHub events.

Examples:

* Code Push
* Pull Request Creation
* Tag Creation
* Release Creation
* Scheduled Execution
* Manual Execution

Workflows are defined using YAML files stored in:

```text
.github/workflows/
```

Example:

```yaml
name: Build Application

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Build
        run: make
```

---

# Why Do We Need GitHub Actions?

In traditional development:

```text
Developer Pushes Code
        ↓
Developer Builds Manually
        ↓
Developer Runs Tests
        ↓
Developer Deploys Application
```

Problems:

* Human errors
* Repetitive tasks
* Slow delivery
* Inconsistent builds
* Deployment mistakes

GitHub Actions automates these tasks.

```text
Developer Pushes Code
        ↓
GitHub Actions Triggered
        ↓
Build
        ↓
Test
        ↓
Package
        ↓
Deploy
```

Benefits:

* Faster development
* Automated testing
* Consistent deployments
* Improved code quality
* Reduced manual effort

---

# Key Concepts

## Workflow

A workflow is an automated process.

Example:

```yaml
name: CI Pipeline
```

A workflow contains one or more jobs.

---

## Event

An event triggers a workflow.

Examples:

```yaml
on:
  push:
```

```yaml
on:
  pull_request:
```

```yaml
on:
  schedule:
```

---

## Job

A job is a group of steps executed on a runner.

Example:

```yaml
jobs:
  build:
```

---

## Step

A step performs a specific task.

Example:

```yaml
- name: Run Tests
  run: pytest
```

---

## Action

Reusable functionality.

Example:

```yaml
uses: actions/checkout@v4
```

---

## Runner

A machine that executes workflows.

Examples:

* Ubuntu Runner
* Windows Runner
* macOS Runner
* Self-hosted Runner

---

# Workflow Architecture

```text
GitHub Event
      ↓
Workflow Trigger
      ↓
Runner
      ↓
Jobs
      ↓
Steps
      ↓
Results
```

---

# Components of GitHub Actions

## Workflow File

Stored in:

```text
.github/workflows/
```

Example:

```text
.github/workflows/build.yml
```

---

## Jobs

Example:

```yaml
jobs:
  build:
  test:
  deploy:
```

Jobs can run:

* Sequentially
* In parallel

---

## Steps

Example:

```yaml
steps:
  - checkout code
  - install dependencies
  - build project
  - run tests
```

---

## Marketplace Actions

GitHub provides reusable actions.

Examples:

* Checkout Code
* Setup Python
* Setup Node.js
* Docker Build
* AWS Deployment

---

# Workflow Lifecycle

```text
Code Push
    ↓
Workflow Trigger
    ↓
Checkout Repository
    ↓
Install Dependencies
    ↓
Build Project
    ↓
Run Tests
    ↓
Generate Artifacts
    ↓
Deploy
```

---

# Common Use Cases

## Continuous Integration (CI)

Automatically:

* Build application
* Run tests
* Verify code quality

Example:

```yaml
on:
  pull_request:
```

---

## Continuous Deployment (CD)

Automatically deploy:

* Web applications
* APIs
* Containers
* Cloud applications

---

## Docker Image Build

```yaml
docker build
docker push
```

---

## Embedded Linux / Yocto

Automate:

* Yocto builds
* Image generation
* Package validation
* Static analysis

---

## Security Scanning

Run:

* Dependency checks
* Vulnerability scans
* Secret detection

---

## Scheduled Tasks

Example:

```yaml
on:
  schedule:
```

Run nightly builds automatically.

---

# Advantages

## Automation

Reduces manual work.

---

## Built Into GitHub

No separate CI server required.

---

## Faster Development

Immediate feedback after code changes.

---

## Better Code Quality

Tests run automatically.

---

## Easy Integration

Works with:

* Docker
* AWS
* Azure
* Kubernetes
* Yocto
* Python
* Java
* Node.js

---

## Scalable

Supports large projects and teams.

---

# Disadvantages

## YAML Learning Curve

Complex workflows can be difficult initially.

---

## Usage Limits

Free-tier execution limits exist.

---

## Debugging Complexity

Large workflows can be difficult to troubleshoot.

---

## Vendor Dependency

Tightly integrated with GitHub.

---

# Best Practices

* Keep workflows simple
* Separate build and deployment jobs
* Use reusable actions
* Store secrets in GitHub Secrets
* Cache dependencies
* Run tests on pull requests
* Use branch protection rules
* Version control workflow files

---

# Interview Questions

## What is GitHub Actions?

GitHub Actions is GitHub's CI/CD platform used to automate software workflows such as building, testing, and deploying applications.

---

## Why do we need GitHub Actions?

GitHub Actions automates repetitive development tasks such as builds, tests, and deployments, reducing manual effort, improving consistency, and accelerating software delivery.

---

## What is a Workflow?

A workflow is an automated process defined in a YAML file that runs when specific events occur.

---

## What is a Runner?

A runner is the machine that executes workflow jobs.

Examples:

* Ubuntu
* Windows
* macOS
* Self-hosted systems

---

## What is the difference between a Job and a Step?

### Job

A collection of steps executed on a runner.

### Step

An individual task inside a job.

---

## Where are workflow files stored?

```text
.github/workflows/
```

---

## What events can trigger a workflow?

Examples:

* push
* pull_request
* release
* schedule
* workflow_dispatch

---

## Can GitHub Actions be used for Yocto builds?

Yes.

GitHub Actions can automate:

* Yocto image builds
* Package generation
* Testing
* Artifact publishing

---

# Interview Answer (Most Asked)

### Why do we need GitHub Actions?

GitHub Actions is used to automate software development workflows such as building, testing, and deploying applications. It eliminates manual tasks, ensures consistent builds, improves code quality through automated testing, and enables faster software delivery using CI/CD pipelines.

---

# Conclusion

GitHub Actions is a powerful CI/CD automation platform integrated directly into GitHub. It enables teams to automate builds, tests, deployments, security scans, and many other development tasks. By reducing manual intervention and providing automated workflows, GitHub Actions improves software quality, reliability, and development speed.

### 30-Second Interview Answer

**Q: Why do we need GitHub Actions?**

> GitHub Actions is used to automate the software development lifecycle. Whenever code is pushed or a pull request is created, it can automatically build the project, run tests, perform code quality checks, and deploy the application. This reduces manual effort, minimizes human errors, ensures consistent builds, and enables faster and more reliable software delivery through CI/CD practices.
