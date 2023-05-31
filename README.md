# Forking Test

This purpose of this repository is to test the behavior of GitHub Actions with
repository forks, pull requests, etc.

## Relevant Settings

At the enterprise, organization, and repository levels, there are certain
settings that will affect the behavior of GitHub Actions for repository forks.

- Fork pull request workflows from outside collaborators
  - Require approval for first-time contributors who are new to GitHub
  - Require approval for first-time contributors
  - Require approval for all outside collaborators
- Fork pull request workflows in private repositories
  - This tells Actions to run workflows from pull requests originating from
    repository forks. Note that doing so will give maintainers of those forks
    the ability to use tokens with read permissions on the source repository.

When creating workflow files, the type of event that triggers the workflow can
also influence when it will appear in the UI. For more information, see
[Events that trigger workflows](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows).
Specifically, if the event has the following note, it will only show in the UI
when it is on the default branch.

```plain
Note: This event will only trigger a workflow run if the workflow file is on the default branch.
```

## Testing the Actions UI Tab

The Actions UI has varying behavior for when new or updated actions are visible.
The goal of this testing is to determine when exactly a workflow will appear in
the UI.

High-level findings from the below tests are:

1. Workflows that are in the **default** branch of a repository are visible in
   the Actions tab of the GitHub UI
2. If a workflow exists on the **default** branch of a repository, and is
   modified via a PR from a fork, the workflow present in the Actions tab of the
   GitHub UI is one from the default branch, not the fork.
3. If the workflow is in a **non-default** branch and only defines event types
   that are only triggered if the file is in the default branch, it will not
   appear in the GitHub UI.
4. If the workflow is in a **non-default** branch and defines event types that
   are **not** only triggered if the file is in the default branch, it will
   appear in the GitHub UI the **first** time it is invoked.

### Test 1

This test simply pushes a new workflow file to the `main` branch. The
`00-pr-comment.yml` workflow simply dumps the event context and adds a comment
to newly-created PRs using the `pull_request` event trigger.

- **Source Repo:** `ncalteen-actions/forking-test`
- **Source Branch:** `main`
- **Activity:** Push new file, `00-pr-comment.yml`, directly to `main`
- **Result:**
  - Workflow appears immediately in the Actions tab
  - Workflow does not run

### Test 2

> **Note:** By default, when a repository that contains workflows is forked, all
> workflows are initially disabled.

- **Source Repo:** `ncalteen-testuser/forking-test`
- **Source Branch:** `main`
- **Activity:** Fork `ncalteen-actions/forking-test` to
  `ncalteen-testuser/forking-test`
- **Result:**
  - Workflow is already present in the Actions tab

### Test 3

- **Source Repo:** `ncalteen-testuser/forking-test`
- **Source Branch:** `main`
- **Activity:**
  - Update `00-pr-comment.yml` and push directly to `main`
  - Submit PR to `ncalteen-actions/forking-test`
- **Result:**
  - Workflow is already present in the Actions tab
  - Workflow run must be approved by a maintainer

### Test 4

- **Source Repo:** `ncalteen-testuser/forking-test`
- **Source Branch:** `main`
- **Activity:**
  - Create `01-context.yml` and push directly to `main`
  - Submit PR to `ncalteen-actions/forking-test`
- **Result:**
  - Workflow is immediately present in the Actions tab of the **forked**
    repository
  - Workflow is **not** immediately present in the Actions tab of the **source**
    repository
- **Activity:**
  - Approve and merge PR into `ncalteen-actions/forking-test`
- **Result:**
  - Workflow is now present in the Actions tab of the **source**

### Test 5

This new workflow file triggers on `workflow_dispatch` event type, which can
only be triggered if the workflow file exists on the default branch.

- **Source Repo:** `ncalteen-actions/forking-test`
- **Source Branch:** `new-workflow`
- **Activity:**
  - Create `02-another-context-workflow.yml` and push to `new-workflow`
- **Result:**
  - Workflow is **not** present in the Actions tab of the repository
- **Activity:**
  - Invoke `02-another-context-workflow.yml` in the `new-workflow` branch via
    the GitHub API (see below cURL command)
- **Result:**
  - A `404` error is thrown.

```bash
curl -L \
  -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer $GITHUB_TOKEN"\
  -H "X-GitHub-Api-Version: 2022-11-28" \
  https://api.github.com/repos/ncalteen-actions/forking-test/actions/workflows/02-another-context-workflow.yml/dispatches \
  -d '{"ref":"new-workflow"}'
```

### Test 6

This new workflow file triggers on `workflow_dispatch` and `push` event types
(the `push` event type can be triggered even if the file does not exist in the
default branch).

- **Source Repo:** `ncalteen-actions/forking-test`
- **Source Branch:** `new-workflow`
- **Activity:**
  - Create `02-another-context-workflow.yml` and push to `new-workflow`
- **Result:**
  - Workflow is **not** present in the Actions tab of the repository
- **Activity:**
  - Invoke `02-another-context-workflow.yml` in the `new-workflow` branch via
    the GitHub API (see below cURL command)
- **Result:**
  - Workflow is present in the Actions tab of the repository

```bash
curl -L \
  -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer $GITHUB_TOKEN"\
  -H "X-GitHub-Api-Version: 2022-11-28" \
  https://api.github.com/repos/ncalteen-actions/forking-test/actions/workflows/02-another-context-workflow.yml/dispatches \
  -d '{"ref":"new-workflow"}'
```
