<h2 align="center">
    <a href="https://nullplatform.com" target="blank_">
        <img height="100" alt="nullplatform" src="https://nullplatform.com/favicon/android-chrome-192x192.png" />
    </a>
    <br>
    <br>
    Setup CI GitHub Actions Suite
    <br>
</h2>

## Overview

The "Setup CI" GitHub Actions suite provides a set of actions designed to automate the CI setup for nullplatform applications. It simplifies the process of configuring CI/CD pipelines for single and multiple applications (monorepo) within your GitHub Actions workflows.

## Table of Contents

- [Inputs](#inputs)
- [Outputs](#outputs)
- [Usage](#usage)
  - [Use Case: Setup CI for a Single Application](#use-case-setup-ci-for-a-single-application)
  - [Use Case: Setup CI for Multiple Applications (Monorepo)](#use-case-setup-ci-for-multiple-applications-monorepo)
- [License](#license)

## Inputs

### `api-key`

- **Description**: The API key required for authentication. (Optional)
- **Required**: No

### `status`

- **Description**: The status to set for the CI setup. (Optional, default: `pending`)
- **Required**: No

### `monorepo`

- **Description**: Indicates if the repository follows a monorepo structure. (Optional, default: `false`)
- **Required**: No

### `applications-path`

- **Description**: The path to the applications within the repository. (Optional)
- **Required**: No

### `application-id`

- **Description**: The ID of the application being set up. (Optional)
- **Required**: No

## Outputs

If `monorepo` flag is set to `false`

### `application-id`

- **Description**: The ID of the application being set up.

### `application-name`

- **Description**: The name of the application being set up.

### `application-path`

- **Description**: The path to the applications within the repository.

### `build-id`

- **Description**: The ID of the build launched by the CI.

Otherwise, if `monorepo` flag is set to `true`

### `applications`

- **Description**: The applications being set up.

## Usage

Here are common use cases for the "Setup CI" GitHub Actions suite:

### Use Case: Setup CI for a Single Application

```yaml
name: Setup CI for Single Application

on:
  push:
    branches:
      - main

permissions:
  id-token: write
  contents: read
  packages: read

jobs:
  setup-ci-single:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Start Nullplatform CI
        id: setup-ci
        uses: nullplatform/github-action-setup-ci@v1
        with:
          api-key: ${{ secrets.NULLPLATFORM_API_KEY }}

      ##
      ## Other CI/CD steps
      ## 

      - name: End Nullplatform CI
        if: ${{ always() }}
        id: end-setup-ci
        uses: nullplatform/github-action-setup-ci@main
        with:
          build-id: ${{ steps.setup-ci.outputs.build-id }}
          status: ${{ contains(fromJSON('["failure", "cancelled"]'), job.status) && 'failed' || 'successful' }}
```

In this example, the GitHub Action sets up CI for a single nullplatform application, including the "Start Nullplatform CI" and "End Nullplatform CI" steps.

### Use Case: Setup CI for Multiple Applications (Monorepo)

```yaml
name: Setup CI for Multiple Applications (Monorepo)
on:
  push:
    branches:
      - main

permissions:
  id-token: write
  contents: read
  packages: read

jobs:
  fetch-data:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        id: checkout-code
        uses: actions/checkout@v4
        with:
          fetch-depth: 2
      - name: Get changed applications
        id: get-changed-applications
        run: |
          changed_applications=$(git diff --name-only HEAD^ HEAD -- apps | cut -d / -f 1,2 | uniq | paste -sd "," -)
          echo "Changed applications: $changed_applications"
          echo "applications=$changed_applications" >> $GITHUB_OUTPUT
      - name: Start Nullplatform CI for changed applications
        id: nullplatform-ci-monorepo
        uses: nullplatform/github-action-setup-ci@v1
        with:
          api-key: ${{ secrets.NULLPLATFORM_API_KEY }}
          monorepo: true
          applications-path: ${{ steps.get-changed-applications.outputs.applications }}
    outputs:
      metadata: ${{ steps.nullplatform-ci-monorepo.outputs.applications }}
  build:
    needs: fetch-data
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        application: ${{ fromJson(needs.fetch-data.outputs.metadata) }}
    steps:
      - name: Start Nullplatform CI
        id: nullplatform-ci
        uses: nullplatform/github-action-setup-ci@v1
        with:
          api-key: ${{ secrets.NULLPLATFORM_API_KEY }}
          application-id: ${{ matrix.application.id }}

      ##
      ## Other steps
      ##

      - name: End Nullplatform CI
        if: ${{ always() }}
        id: end-nullplatform-ci
        uses: nullplatform/github-action-setup-ci@v1
        with:
          build-id: ${{ steps.nullplatform-ci.outputs.build-id }}
          status: ${{ contains(fromJSON('["failure", "cancelled"]'), job.status) && 'failed' || 'successful' }}
```

In this example, the GitHub Action sets up CI for multiple nullplatform applications within a monorepo structure, including the "Start Nullplatform CI" and "End Nullplatform CI" steps. You specify the `applications-path` to indicate the location of the applications.

## License

This GitHub Action is licensed under the [MIT License](LICENSE).
