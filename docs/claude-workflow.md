# Claude Code Workflow

## Overview

This GitHub Workflow, named "Claude Code," enables interaction with Claude Code based on specific GitHub events such as issue comments, pull request review comments, new issues, or pull request reviews. It triggers only when a specific mention (`@claude`) is found in the relevant text content. The workflow then uses the `anthropics/claude-code-action` to process the context.

## Inputs

This workflow does not define any inputs using `workflow_dispatch` or `inputs` within `on:`. It is triggered automatically by repository events.

## Outputs

This workflow does not produce any formal outputs. The primary interaction and output would be managed by the `anthropics/claude-code-action`, which typically involves Claude responding to the comment or issue where it was mentioned.

## Secrets

| Name                | Description                                               | Required |
|---------------------|-----------------------------------------------------------|----------|
| `ANTHROPIC_API_KEY` | Passed to the `anthropics/claude-code-action@beta`.       | Yes      |

## Environment Variables

This workflow does not explicitly define or use custom environment variables at the workflow level.

## Usage

This workflow is designed to run automatically when specific events occur and the `@claude` mention is present.

To enable this workflow:
1.  Ensure the `.github/workflows/claude.yml` file exists in the repository.
2.  Add the `ANTHROPIC_API_KEY` secret to the repository or organization GitHub secrets.

The workflow will then trigger automatically based on the defined events and conditions. Users can invoke Claude by mentioning `@claude` in:
-   Issue comments
-   Pull request review comments
-   The body of a pull request review
-   The body or title of a new issue

## Key Steps/Jobs

The workflow consists of a single job: `claude`.

### Job: `claude`

-   **Conditional Execution (`if`):**
    The job only runs if the `@claude` mention is present in the relevant context of the triggering event:
    -   For `issue_comment`: in `github.event.comment.body`.
    -   For `pull_request_review_comment`: in `github.event.comment.body`.
    -   For `pull_request_review`: in `github.event.review.body`.
    -   For `issues`: in `github.event.issue.body` or `github.event.issue.title`.

-   **Runs on:** `ubuntu-latest`

-   **Permissions:**
    -   `contents: read` - Allows checking out the repository.
    -   `pull-requests: read` - Allows the action to read pull request data.
    -   `issues: read` - Allows the action to read issue data.
    -   `id-token: write` - Allows the action to potentially authenticate to cloud services if needed by the underlying `claude-code-action` for specific tools or features (though not explicitly used by the basic setup shown).

-   **Steps:**

    1.  **Checkout repository:**
        -   `uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`
        -   `with: fetch-depth: 1`
        -   Checks out the repository's code. `fetch-depth: 1` gets only the latest commit.

    2.  **Run Claude Code:**
        -   `id: claude`
        -   `uses: anthropics/claude-code-action@beta`
        -   Invokes the official Claude Code action from Anthropics.
        -   `with:`
            -   `anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}`: Provides the Anthropic API key to the action.

## Dependencies

-   **`actions/checkout@v4`**: Used to checkout repository code.
-   **`anthropics/claude-code-action@beta`**: The core GitHub Action from Anthropics that provides Claude Code functionality. This workflow serves as a trigger and environment setup for this action.
-   The GitHub Actions environment itself.

## Triggers

-   **`on:`**
    -   `issue_comment:`
        -   `types: [created]` (when a new issue comment is created)
    -   `pull_request_review_comment:`
        -   `types: [created]` (when a new pull request review comment is created)
    -   `issues:`
        -   `types: [opened, assigned]` (when an issue is opened or assigned)
    -   `pull_request_review:`
        -   `types: [submitted]` (when a pull request review is submitted)

The job will only proceed if the conditional `if` statement (checking for `@claude`) evaluates to true for these events.
