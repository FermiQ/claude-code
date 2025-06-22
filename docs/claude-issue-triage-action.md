# Claude Issue Triage Action

## Overview

This GitHub Action automates the triage of GitHub issues using Claude Code. When triggered (typically on new issues), it uses Claude Code to analyze the issue content, determine appropriate labels from the repository's available labels, and apply them. The action is designed to only apply labels and explicitly avoids posting comments.

## Inputs

| Name                | Description                                          | Default | Required |
|---------------------|------------------------------------------------------|---------|----------|
| `timeout_minutes`   | Timeout in minutes for the entire action's execution. | `"5"`   | No       |
| `anthropic_api_key` | Anthropic API key for Claude Code.                   |         | Yes      |
| `github_token`      | GitHub token with repo and issues permissions.       |         | Yes      |

## Outputs

This Action does not produce any formal outputs. Its primary effect is the application of labels to the GitHub issue that triggered the workflow.

## Secrets

| Name                  | Description                                                                | Required |
|-----------------------|----------------------------------------------------------------------------|----------|
| `ANTHROPIC_API_KEY`   | Passed to the underlying `claude-code-action` via `inputs.anthropic_api_key`. This is your Anthropic API key. | Yes      |
| `GITHUB_TOKEN`        | Passed to the underlying `claude-code-action` via `inputs.github_token`. This is typically `${{ secrets.GITHUB_TOKEN }}` and is used by Claude to interact with the GitHub API (e.g., list labels, get issue details, update issues). | Yes      |

## Environment Variables

This action itself does not define specific environment variables for its top-level operation, but it relies on standard GitHub Actions environment variables (like `github.repository`, `github.event.issue.number`) within the prompt construction. The `claude-code-action` it calls will use environment variables as documented for that action.

## Usage

This action is typically used in a workflow that triggers on new issues.

```yaml
name: Issue Triage with Claude

on:
  issues:
    types: [opened, reopened] # Or other relevant types

jobs:
  triage_issue:
    runs-on: ubuntu-latest
    permissions:
      issues: write # Required to allow the action to label issues
      contents: read # Required for actions/checkout
    steps:
      - name: Run Claude Issue Triage
        uses: ./.github/actions/claude-issue-triage-action # Or FermiQ/claude-code/.github/actions/claude-issue-triage-action@main
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
          # timeout_minutes: "7" # Optional: override default timeout
```

## Key Steps/Jobs

This is a composite action with the following main execution steps:

1.  **Checkout repository code:**
    *   Uses `actions/checkout@v4` with `fetch-depth: 0`. This step is included likely to ensure local actions can be resolved, although this specific action might not directly need the full repo content for its prompt.

2.  **Create prompt file:**
    *   A detailed prompt for Claude Code is dynamically created at `/tmp/claude-prompts/claude-issue-triage-prompt.txt`.
    *   The prompt instructs Claude to act as an issue triage assistant.
    *   It includes placeholders for `github.repository` and `github.event.issue.number` to provide context to Claude.
    *   The prompt outlines a specific process for Claude:
        1.  Fetch available repository labels using `gh label list`.
        2.  Use GitHub tools (MCPs - Meta-Code Processors) to get issue details (`mcp__github__get_issue`), comments (`mcp__github__get_issue_comments`), search for similar issues (`mcp__github__search_issues`), and list other issues (`mcp__github__list_issues`).
        3.  Analyze the issue content (title, description, type, severity, etc.).
        4.  Select appropriate labels from the fetched list.
        5.  Apply selected labels using `mcp__github__update_issue`.
    *   Crucially, the prompt emphasizes **not** to post comments and that the **only** action should be applying labels.

3.  **Run Claude Code:**
    *   This step calls the local `claude-code-action` (defined in `../claude-code-action/action.yml`).
    *   `prompt_file`: Passes the path to the dynamically created prompt file.
    *   `allowed_tools`: Specifies the tools Claude is permitted to use:
        *   `Bash(gh label list)`: Allows running the `gh label list` command.
        *   `mcp__github__get_issue`
        *   `mcp__github__get_issue_comments`
        *   `mcp__github__update_issue`
        *   `mcp__github__search_issues`
        *   `mcp__github__list_issues`
    *   `install_github_mcp: "true"`: Ensures the GitHub MCP server is installed and available for Claude.
    *   `timeout_minutes`: Passed from the input.
    *   `anthropic_api_key`: Passed from the input.
    *   `github_token`: Passed from the input.

## Dependencies

-   **`actions/checkout@v4`**: Used to checkout repository code.
-   **`./.github/actions/claude-code-action`**: This action is a primary dependency, as it orchestrates the actual interaction with the Claude Code API.
-   **`gh` CLI (GitHub CLI)**: Implied dependency, as the prompt instructs Claude to use `gh label list`. This must be available in the runner environment.
-   **GitHub MCP Server**: Required by the `claude-code-action` when `install_github_mcp` is true, for Claude to interact with GitHub resources via tools.
-   Relies on the standard GitHub Actions runner environment and its built-in capabilities.
