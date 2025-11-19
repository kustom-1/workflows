# Reusable GitHub Actions Workflows

This repository contains reusable workflows for standard CI/CD processes, designed to be called from other GitHub Actions workflows.

## Backend Deployment to AWS

The `upload-artifact.yaml` workflow provides a standardized way to build, package, and deploy a Node.js backend application to an AWS environment using an Auto Scaling Group (ASG). It also includes special handling for Next.js applications.

### Key Features

-   **Reusable:** Designed to be called from any other workflow using `workflow_call`.
-   **Automated Build:** Checks out code, sets up Node.js, and runs build commands.
-   **Artifact Management:** Packages the application into a versioned ZIP file and uploads it to a dedicated S3 bucket.
-   **Zero-Downtime Deployments:** Triggers an instance refresh on a specified Auto Scaling Group to roll out the new version safely.
-   **Next.js Support:** Includes an optional step to sync static assets (`.next/static` and other specified paths) to a separate S3 bucket for serving via CloudFront.
-   **Configurable:** Uses a comprehensive set of inputs to adapt to different project structures and deployment needs.

### Usage

To use this workflow, you need to call it from a workflow in your own repository.

**Example `.github/workflows/deploy.yml`:**

```yaml
name: Deploy to Staging

on:
  push:
    branches:
      - main

jobs:
  deploy-backend:
    uses: your-organization/your-repo/.github/workflows/upload-artifact.yaml@main
    with:
      environment: 'staging'
      version_tag: 'v1.0.0'
      app_name: 'my-cool-app'
      working_directory: './backend'
      # Set to true if you are deploying a Next.js application
      isNext: true
      static_assets_bucket_name: 'my-cloudfront-assets-prefix'
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      JWT_SECRET: ${{ secrets.MY_APP_JWT_SECRET }}
```

### Inputs

The workflow is highly customizable through the following inputs:

| Input                      | Description                                                              | Type      | Required | Default                |
| -------------------------- | ------------------------------------------------------------------------ | --------- | -------- | ---------------------- |
| `environment`              | The deployment environment (e.g., `staging`, `production`).              | `string`  | `true`   |                        |
| `version_tag`              | The version tag for the artifact (e.g., `v1.2.3`).                       | `string`  | `false`  | `latest`               |
| `aws_region`               | The AWS region for the deployment.                                       | `string`  | `false`  | `us-east-1`            |
| `node_version`             | The Node.js version to use for the build.                                | `string`  | `false`  | `20.x`                 |
| `app_name`                 | The name of the application.                                             | `string`  | `true`   |                        |
| `build_command`            | The command to build the application.                                    | `string`  | `false`  | `npm run build`        |
| `artifact_path`            | The path to the built artifacts (e.g., `dist`, `.next`).                 | `string`  | `false`  | `dist`                 |
| `s3_artifact_prefix`       | The prefix for the artifact in the S3 bucket (e.g., `backend`).          | `string`  | `false`  | `backend`              |
| `asg_suffix`               | The suffix of the Auto Scaling Group name (e.g., `application-asg`).     | `string`  | `false`  | `application-asg`      |
| `working_directory`        | The directory where the application code is located.                     | `string`  | `false`  | `.`                    |
| `health_check_path`        | The path for the application's health check endpoint.                    | `string`  | `false`  | `/health`              |
| `log_path`                 | The path to the application logs on the instance.                        | `string`  | `false`  | `/var/log/app/`        |
| `install_command`          | The command to install dependencies.                                     | `string`  | `false`  | `npm ci`               |
| `min_healthy_percentage`   | The minimum percentage of healthy instances during an ASG refresh.       | `number`  | `false`  | `50`                   |
| `instance_warmup`          | The instance warmup time in seconds during an ASG refresh.               | `number`  | `false`  | `300`                  |
| `isNext`                   | Set to `true` if deploying a Next.js application.                        | `boolean` | `false`  | `false`                |
| `static_assets_bucket_name`| The prefix for the S3 bucket used for CloudFront static assets.          | `string`  | `false`  | `''`                   |

### Secrets

The following secrets must be provided to the workflow for it to authenticate with AWS and other services.

| Secret                    | Description                                          | Required |
| ------------------------- | ---------------------------------------------------- | -------- |
| `AWS_ACCESS_KEY_ID`       | The AWS access key ID for deployment.                | `true`   |
| `AWS_SECRET_ACCESS_KEY`   | The AWS secret access key for deployment.            | `true`   |
| `JWT_SECRET`              | A JWT secret, passed as an environment variable during the build. | `false`  |

### Outputs

The workflow produces the following outputs, which can be used in subsequent jobs.

| Output         | Description                                  |
| -------------- | -------------------------------------------- |
| `artifact_url` | The S3 URL of the uploaded application artifact. |
| `refresh_id`   | The ID of the triggered ASG instance refresh.    |
