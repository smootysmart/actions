# actions

-----

# Reusable Workflows Documentation

This repository contains a set of reusable GitHub Actions workflows designed to standardize and simplify the CI/CD processes for the LSEG Immersion Day DevOps Workshop. These workflows can be called from any other repository to build, test, scan, and deploy applications.

-----

## workflows Available

### 1\. Reusable Application CI Check

**File:** `build-test-ci.yml`

This workflow performs a standard Continuous Integration (CI) check for a Node.js application. It validates code changes by building the application and running its test suite. The workflow will fail if any step does not complete successfully.

**Key Steps:**

  * Checks out the source code.
  * Sets up a **Node.js 20** environment.
  * Installs dependencies with `npm install`.
  * Verifies the production build with `npm run build`.
  * Executes the test suite with `npm test`.

**Inputs & Secrets:**

  * This workflow has **no inputs**.
  * This workflow requires **no secrets**.

<br>

### 2\. Reusable Trivy Security Scan 🛡️

**File:** `security-scan-ci.yml`

This workflow provides an automated security vulnerability scan for Docker images using Trivy. It builds the Docker image locally (without pushing it to a registry) and scans it for known vulnerabilities.

The workflow is configured with a strict policy: it will **fail** if any **`CRITICAL`** or **`HIGH`** severity vulnerabilities are discovered.

**Key Steps:**

  * Builds the Docker image locally using the `load: true` flag.
  * Scans the locally built image with Trivy.
  * Fails the job if critical or high-severity issues are found.

**Inputs & Secrets:**

  * This workflow has **no inputs**.
  * This workflow requires **no secrets**.

<br>

### 3\. Reusable Docker Push

**File:** `docker-push-ci.yml`

This workflow builds a Docker image from a `Dockerfile` and pushes it to Docker Hub. It streamlines the deployment process by encapsulating the login, build, and push logic.

**Inputs:**

| Input | Required | Description |
| :--- | :--- | :--- |
| `image_name` | **Yes** | The name of the Docker image (e.g., `your-username/app-name`). |
| `tag` | **Yes** | The tag for the Docker image (e.g., `latest` or a Git SHA). |

**Secrets:**

You must configure the following secrets in the repository *using* this workflow (**Settings \> Secrets and variables \> Actions**):

| Secret | Required | Description |
| :--- | :--- | :--- |
| `DOCKER_USERNAME` | **Yes** | Your Docker Hub username. |
| `DOCKER_PASSWORD` | **Yes** | Your Docker Hub password or Access Token. |

-----

## Complete Pipeline Example

Here is an example of a complete CI/CD pipeline that chains all three reusable workflows together. This workflow ensures that code is built, tested, and scanned for vulnerabilities *before* it is pushed to Docker Hub.

Create this file in your application repository at `.github/workflows/main-pipeline.yml`.

```yaml
name: Full CI, Scan, and Deploy Pipeline

on:
  push:
    branches: [ main ]

jobs:
  # 1. First, run the build and test checks
  run-ci-checks:
    uses: LSEG-Immersion-Day-DevOps-workshop-2025/actions/.github/workflows/build-test-ci.yml@main

  # 2. If CI passes, run the security scan
  run-security-scan:
    needs: run-ci-checks
    uses: LSEG-Immersion-Day-DevOps-workshop-2025/actions/.github/workflows/security-scan-ci.yml@main

  # 3. If the security scan passes, push the image to Docker Hub
  build-and-push-image:
    needs: run-security-scan
    uses: LSEG-Immersion-Day-DevOps-workshop-2025/actions/.github/workflows/docker-push-ci.yml@main
    with:
      # Replace 'your-dockerhub-username' with your actual Docker Hub username
      image_name: your-dockerhub-username/trip-planner-frontend
      tag: ${{ github.sha }}
    secrets:
      # These secrets must be created in your application repository's settings
      DOCKER_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKER_PASSWORD: ${{ secrets.DOCKERHUB_TOKEN }}
```
