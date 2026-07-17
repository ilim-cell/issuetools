# Customizable Maintainer Issue Commands

Automate issue triage workflows using fully customizable text commands in issue comments. This composite GitHub Action allows repository maintainers to close, reopen, or mark issues as duplicates directly from the comment interface.

## Features

* Close issues as completed or not planned.
* Reopen previously closed issues.
* Mark issues as duplicates using either an issue number or an exact title match.
* Full control over command syntax strings.
* Configurable text templates for automated confirmation and error responses.

## Setup Instructions

### 1. Create the Workflow File
In the repository where you want to use the commands, create a new file named `.github/workflows/triage.yml`.

### 2. Add the Configuration
Copy and paste the configuration below into your newly created workflow file. 

> [!IMPORTANT]
> Make sure to keep the `permissions` block intact so the action has the necessary rights to modify issues. 

```yaml
name: Triage

on:
  issue_comment:
    types: [created]

# Required permissions for the GITHUB_TOKEN to modify issues
permissions:
  issues: write

jobs:
  handle-commands:
    if: |
      github.event.comment.author_association == 'OWNER' || 
      github.event.comment.author_association == 'MEMBER' || 
      github.event.comment.author_association == 'COLLABORATOR'
    runs-on: ubuntu-latest

    steps:
      - name: Run Triage Commands Action
        uses: ilim-cell/issuetools@v1
        with:
          github_token: \${{ secrets.GITHUB_TOKEN }}
          cmd_close_completed: '/close completed'
          cmd_close_not_planned: '/close not planned'
          cmd_reopen: '/reopen'
          cmd_dup_prefix: '/dup'
```

## Configuration Inputs

The following parameters must be configured in the `with` block of your workflow:

| Input | Description | Required | Default |
| :--- | :--- | :--- | :--- |
| `github_token` | The token used to interact with the GitHub API. | Yes | `${{ github.token }}` |
| `cmd_close_completed` | Command string to close an issue as completed. | Yes | `/close completed` |
| `cmd_close_not_planned` | Command string to close an issue as not planned. | Yes | `/close not planned` |
| `cmd_reopen` | Command string to reopen an issue. | Yes | `/reopen` |
| `cmd_dup_prefix` | Prefix string for duplicate commands. | Yes | `/dup` |

The following parameter configurations are optional. If left blank, the action uses standard system strings:

| Input | Description | Required | Default |
| :--- | :--- | :--- | :--- |
| `msg_dup_not_found` | Error string when duplicate target cannot be found. Supports `{input}` token. | No | System default error text. |
| `msg_dup_missing_issue` | Error string when duplicate number does not exist. Supports `{target}` token. | No | System default error text. |
| `msg_dup_success_comment` | Automated message posted on the closed duplicate issue. Supports `{target}` token. | No | System default success text. |

## Usage Examples

Once deployed, repository maintainers can use the configured strings in any issue comment:

* Type `/close completed` to close the issue as finished.
* Type `/close not planned` to close the issue as skipped or canceled.
* Type `/reopen` to open a closed issue.
* Type `/dup #123` to close the current issue and link it to issue 123.
* Type `/dup "Exact Title of Another Issue"` to find and link the target issue by its text title.

## Troubleshooting

If the action encounters a `403 Forbidden` error when running commands, confirm that:
1. The `permissions: issues: write` block is explicitly defined at the top level of your workflow file.
2. Under repository **Settings** > **Actions** > **General** > **Workflow permissions**, the option **Read and write permissions** is selected.
