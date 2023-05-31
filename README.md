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

## References

- [Approving workflow runs from public forks](https://docs.github.com/en/actions/managing-workflow-runs/approving-workflow-runs-from-public-forks)
