# Claude Issue Triage Workflow

## Overview

This GitHub Workflow, named "Claude Issue Triage," automates the process of triaging newly opened GitHub issues using the `claude-issue-triage-action`. When an issue is opened in the repository, this workflow triggers, invoking the action to analyze the issue and apply appropriate labels.

## Inputs

This workflow does not define any inputs using `workflow_dispatch` or `inputs` within `on:`. It is triggered automatically by repository events.

## Outputs

This workflow does not produce any formal outputs. Its primary effect is the application of labels to GitHub issues by the `claude-issue-triage-action` it runs.

## Secrets

| Name                | Description                                      | Required |
|---------------------|--------------------------------------------------|----------|
| `ANTHROPIC_API_KEY` | Passed to the `claude-issue-triage-action`.      | Yes      |
| `GITHUB_TOKEN`      | Passed to the `claude-issue-triage-action`. This is the standard GitHub Actions token, which has necessary permissions due to the `permissions` block in the job. | Yes      |

## Environment Variables

This workflow does not explicitly define or use custom environment variables at the workflow level.

## Usage

This workflow is designed to run automatically when issues are opened in the repository where it is located. No manual invocation is typically required.

To enable this workflow:
1.  Ensure the `.github/workflows/claude-issue-triage.yml` file exists in the repository.
2.  Ensure the referenced action ` ./.github/actions/claude-issue-triage-action` is present and correctly configured in the same repository.
3.  Add the `ANTHROPIC_API_KEY` secret to the repository or organization GitHub secrets.

The workflow will then trigger automatically whenever a new issue is opened.

## Key Steps/Jobs

The workflow consists of a single job: `triage-issue`.

### Job: `triage-issue`

-   **Runs on:** `ubuntu-latest`
-   **Timeout:** 10 minutes
-   **Permissions:**
    -   `contents: read` - Allows checking out the repository.
    -   `issues: write` - Allows the action to read issue data and apply labels to issues.

-   **Steps:**

    1.  **Checkout repository:**
        -   `uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`
        -   Checks out the repository's code, which is necessary to use the local action (`./.github/actions/claude-issue-triage-action`).

    2.  **Run Claude Issue Triage:**
        -   `uses: ./.github/actions/claude-issue-triage-action`
        -   Invokes the local custom action responsible for the issue triage logic.
        -   `with:`
            -   `anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}`: Provides the Anthropic API key to the action.
            -   `github_token: ${{ secrets.GITHUB_TOKEN }}`: Provides the GitHub token to the action. The token's permissions are defined at the job level.

## Dependencies

-   **`actions/checkout@v4`**: Used to checkout repository code.
-   **`./.github/actions/claude-issue-triage-action`**: The core local GitHub Action that performs the actual triage logic using Claude Code. This workflow serves as a trigger and environment setup for this action.
-   The GitHub Actions environment itself.

## Trigger

-   **`on: issues`**
    -   `types: [opened]`
    -   This workflow is triggered whenever an issue is opened in the repository.
