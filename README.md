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

## Testing the Actions UI Tab

The Actions UI has varying behavior for when new or updated actions are visible.
The goal of this testing is to determine when exactly a workflow will appear in
the UI.

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

_Any new workflow files that are pushed to the default branch of a repository
will appear in the Actions tab of the GitHub UI._

### Test 2

- **Source Repo:** `ncalteen-actions/forking-test`
- **Source Branch:** `new-feature`
- **Activity:** Update `00-pr-comment.yml` and submit PR to `main`
- **Result:**
  - Workflow is already present in the Actions tab
  - Workflow runs on the `new-feature` branch

### Test 3

> **Note:** By default, when a repository that contains workflows is forked, all
> workflows are initially disabled.

- **Source Repo:** `ncalteen-testuser/forking-test`
- **Source Branch:** `main`
- **Activity:** Fork `ncalteen-actions/forking-test` to
  `ncalteen-testuser/forking-test`
- **Result:**
  - Workflow is already present in the Actions tab

### Test 4

- **Source Repo:** `ncalteen-testuser/forking-test`
- **Source Branch:** `main`
- **Activity:**
  - Update `00-pr-comment.yml` and push directly to `main`
  - Submit PR to `ncalteen-actions/forking-test`
- **Result:**
  - Workflow is already present in the Actions tab
  - Workflow run must be approved by a maintainer

## References

- [Approving workflow runs from public forks](https://docs.github.com/en/actions/managing-workflow-runs/approving-workflow-runs-from-public-forks)
