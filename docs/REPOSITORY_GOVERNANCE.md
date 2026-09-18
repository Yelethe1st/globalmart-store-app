# Repository Governance

## Branching Model

GlobalMart uses trunk-based development with short-lived branches.

Examples:

- `feature/ABC-123-description`
- `fix/ABC-123-description`
- `docs/description`
- `hotfix/INC-123-description`

Direct pushes to `main` are prohibited.

## Pull Request Requirements

Normal application changes require:

- pull request
- successful required checks
- required reviewer approval
- all conversations resolved
- no unresolved security findings

## Merge Strategy

Preferred merge strategy: squash merge.

## Governance Phases

Governance is introduced in two phases:

- GitHub governance: enforced now
- AWS release governance: introduced later, when the delivery pipeline exists

### GitHub Governance (Now)

The `main` branch is protected by a GitHub ruleset:

| Setting                             | Value                                    |
| ----------------------------------- | ---------------------------------------- |
| Require pull request before merging | On                                       |
| Required approvals                  | 1                                        |
| Dismiss stale approvals             | On                                       |
| Require review from Code Owners     | On                                       |
| Require conversation resolution     | On                                       |
| Require status checks               | On                                       |
| Require branches to be up to date   | On                                       |
| Block force pushes                  | On                                       |
| Block deletions                     | On                                       |
| Require linear history              | Optional                                 |
| Bypass                              | Repository admins, via pull request only |

Required status checks are the existing GitHub Actions jobs in `.github/workflows/pr.yaml`:

- `Semantic Pull Request`
- `Hooks`
- `Project tests`

Merge methods:

| Method        | Value |
| ------------- | ----- |
| Squash merge  | On    |
| Merge commits | Off   |
| Rebase merge  | Off   |

This keeps `main` clean with one logical commit per pull request.

Code owners are defined in `.github/CODEOWNERS`.

While the repository has a single maintainer, repository admins may bypass the ruleset when merging a pull request, because GitHub does not accept a pull request author's approval of their own pull request. Direct pushes to `main` remain blocked. The bypass should be removed once a second reviewer is available.

### AWS Release Governance (Later)

The following controls are introduced once the AWS CI/CD platform exists:

- production manual approval in CodePipeline
- IAM-based production deploy approvers
- AWS-side deployment gates
- CodeBuild security gates
- cross-account deployment permissions
- regional rollout controls

Until CodeBuild security gates exist, "no unresolved security findings" is checked by reviewers rather than enforced automatically.

## Production Deployment

Merging to `main` does not itself grant production access.

Production delivery is performed through the AWS CI/CD platform:

```
GitHub
→ CodePipeline
→ CodeBuild
→ ECR / artifacts
→ CodeDeploy / CloudFormation
→ AWS workloads
```

## Existing GitHub Actions

The upstream application already contains GitHub Actions workflows.

These are preserved initially.

They will be reviewed and classified into:

- repository validation that should remain in GitHub
- functionality that should move to AWS CodeBuild
- upstream release automation that is no longer needed
- functionality that would duplicate the GlobalMart delivery platform
