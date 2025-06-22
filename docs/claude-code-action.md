# Claude Code Action

## Overview

This GitHub Action allows you to run Claude Code within your GitHub Actions workflows. It can take a prompt directly or from a file, interact with Claude Code, and optionally save the output. It also supports installing a GitHub MCP (Meta-Code Processor) server for more advanced interactions.

## Inputs

| Name                 | Description                                                      | Default   | Required |
|----------------------|------------------------------------------------------------------|-----------|----------|
| `github_token`       | GitHub token with repo and issues permissions.                   |           | Yes      |
| `anthropic_api_key`  | Anthropic API key.                                               |           | Yes      |
| `prompt`             | The prompt to send to Claude Code.                               | `""`      | No       |
| `prompt_file`        | Path to a file containing the prompt to send to Claude Code.     | `""`      | No       |
| `allowed_tools`      | Comma-separated list of allowed tools for Claude Code to use.    | `""`      | No       |
| `output_file`        | File to save Claude Code output to (optional).                   | `""`      | No       |
| `timeout_minutes`    | Timeout in minutes for Claude Code execution.                    | `"10"`    | No       |
| `install_github_mcp` | Whether to install the GitHub MCP server.                        | `"false"` | No       |

*Note: At least one of `prompt` or `prompt_file` must be provided and be non-empty.*

## Outputs

This Action does not define formal outputs using the `outputs:` keyword in `action.yml`. However, if `output_file` is specified, the final result from Claude Code will be saved to that file. The complete JSON stream from Claude is saved to `output.json` in the workspace if `output_file` is used.

## Secrets

| Name                  | Description                                       | Required |
|-----------------------|---------------------------------------------------|----------|
| `ANTHROPIC_API_KEY`   | Used by the `Run Claude Code` step via `inputs.anthropic_api_key`. This is your Anthropic API key. | Yes      |
| `GITHUB_TOKEN`        | Used by the `Run Claude Code` and `Install GitHub MCP Server` steps via `inputs.github_token`. This is typically `${{ secrets.GITHUB_TOKEN }}`. | Yes      |

## Environment Variables

| Name                | Description                                                                 | Example Value (Set via) |
|---------------------|-----------------------------------------------------------------------------|-------------------------|
| `PROMPT_PATH`       | Internal variable holding the path to the actual prompt file to be used.    | `/tmp/claude-action/prompt.txt` or value of `inputs.prompt_file` (Set by `Prepare Prompt File` step) |
| `ANTHROPIC_API_KEY` | Passed to the `claude` CLI execution environment.                           | `${{ inputs.anthropic_api_key }}` (Set in `Run Claude Code` step) |
| `GITHUB_TOKEN`      | Passed to the `claude` CLI and MCP server execution environment.            | `${{ inputs.github_token }}` (Set in `Run Claude Code` and `Install GitHub MCP Server` steps) |

## Usage

```yaml
name: Claude Code Example
on: [push]

jobs:
  run_claude_code:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4 # Or appropriate version

      - name: Run Claude Code Action
        uses: ./.github/actions/claude-code-action # Or FermiQ/claude-code/.github/actions/claude-code-action@main if used from another repo
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: "Write a python script to print 'Hello, World!'"
          # or prompt_file: "./path/to/prompt.txt"
          # output_file: "claude_output.txt"
          # allowed_tools: "tool1,tool2"
          # timeout_minutes: "15"
          # install_github_mcp: "true"
```

## Key Steps/Jobs

This is a composite action with the following main execution steps:

1.  **Install Claude Code:**
    *   Runs `npm install -g @anthropic-ai/claude-code` to make the `claude` CLI available.
2.  **Install GitHub MCP Server (Conditional):**
    *   Runs if `inputs.install_github_mcp` is `'true'`.
    *   Configures the `claude` CLI to use a Dockerized GitHub MCP server.
    *   The MCP server uses `inputs.GITHUB_TOKEN` for its operations.
3.  **Prepare Prompt File:**
    *   Checks that either `inputs.prompt` or `inputs.prompt_file` is provided. Exits with an error if not.
    *   If `inputs.prompt_file` is used, it verifies the file exists.
    *   If `inputs.prompt` is used, it writes the prompt content to a temporary file (`/tmp/claude-action/prompt.txt`).
    *   Verifies the chosen prompt file is not empty.
    *   Sets the `PROMPT_PATH` environment variable for the next step.
4.  **Run Claude Code:**
    *   Constructs arguments for the `claude` CLI, including `--allowedTools` if `inputs.allowed_tools` is provided.
    *   Sets a timeout for the `claude` command based on `inputs.timeout_minutes`.
    *   Executes the `claude` command with the prepared prompt (`cat ${{ env.PROMPT_PATH }}`).
    *   If `inputs.output_file` is *not* specified, output is streamed to the console.
    *   If `inputs.output_file` *is* specified:
        *   The output of `claude` is piped to `tee output.txt` (saving to `output.txt` and printing to console).
        *   `output.txt` (which contains a stream of JSON objects) is processed by `jq -s '.'` into a single JSON array, saved as `output.json`.
        *   The `result` field from the last JSON object in the `output.json` array is extracted using `jq -r '.[-1].result'` and saved to the path specified by `inputs.output_file`.
    *   Environment variables `ANTHROPIC_API_KEY` and `GITHUB_TOKEN` are passed from inputs.

## Dependencies

-   **`@anthropic-ai/claude-code`**: The Node.js package for the Claude Code CLI.
-   **`npm`**: Used to install the Claude Code CLI.
-   **`bash`**: Shell used for executing scripts.
-   **`docker`**: Used if `inputs.install_github_mcp` is `true` to run the GitHub MCP server (`ghcr.io/github/github-mcp-server:sha-ff3036d`).
-   **`timeout`**: Standard Linux utility used to limit the execution time of the `claude` command.
-   **`jq`**: Command-line JSON processor, used if `inputs.output_file` is specified.
-   Relies on standard GitHub Actions runner environment features (e.g., environment variable propagation).
