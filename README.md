# CI/CD Pipeline Documentation

This document describes the CI/CD pipeline setup for this project, which uses GitHub and Jenkins to automate testing and deployment for pull requests (PRs).

## Overview

The pipeline is triggered by pull request events (open, reopen, sync, merge, close) in GitHub. These events trigger a Jenkins multibranch pipeline (one for each repository) via github actions sending a request to Jenkins endpoint. The pipeline manages isolated environments for each PR, complete with temporary domains and unique service ports.

## Pipeline Flow

### 1. Triggering the Pipeline

- **Events**: The pipeline is triggered by:
  - Opening a new PR
  - Reopening a closed PR
  - Synchronizing a PR (new commits pushed)
  - Merging a PR
  - Closing a PR
- **Mechanism**: GitHub sends a request to `[Jenkins_url]/github-webhook`, which triggers a rescan by the Jenkins multibranch pipeline.

### 2. PR Environment Setup

For each active PR (including updated ones):

1. **Docker Compose**:
   - If not already running, `docker compose up --build` is executed to spin up services.
2. **Caddy Domain**:
   - A temporary domain `[pr_name].localhost` (for local testing) is created if it doesn't exist.
3. **Service Ports**:
   - Unique ports are assigned to avoid conflicts:
     - Nats service: `6000 + PR number`
     - Platform services: `3000 + PR number`

### 3. Cleanup for Closed/Deleted PRs

- A separate Jenkins job (using the **MultiBranch Action Triggers Plugin**) handles cleanup:
  - Triggered when a PR is mreged or closed.
  - Filters to only act on `PR-*` pipelines.
  - Executes `docker compose down` to remove all services.
  - Removes the associated Caddy domain (`[pr_name].localhost`).

## Requirements

- Jenkins with:
  - GitHub plugin
  - Docker plugin
  - MultiBranch Action Triggers Plugin
- Docker and Docker Compose
- Caddy server
