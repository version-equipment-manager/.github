# Contributing to VEM
 
Welcome. This document describes how we work in our repositories. Following it keeps the history clean, makes releases reproducible, and avoids stepping on each other's toes.
 
If anything here is unclear or out of date, open a PR against this file.
 
---
 
## TL;DR
 
- `main` is always deployable. Never push to it directly.
- Branch off `main`, do your work, open a PR, get one approval, squash merge.
- **PR titles** follow [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) — this is or will be enforced by CI and blocks merge if not met.
- Individual commits on your branch are free-form — write whatever helps you work.
- Branches are named `<type>/<short-description>`.

---
 
## Branching strategy: Release Flow
 
VEM uses trunk-based development with release branches, commonly known as *Release Flow*.
 
### Branches
 
| Branch | Purpose | Lifetime |
|---|---|---|
| `main` | Integration branch. Always deployable. All feature and fix branches merge here. | Permanent |
| `feat/*` | New features | Short-lived |
| `fix/*` | Bug fixes | Short-lived |
| `hotfix/*` | Urgent fixes for active release branches | Short-lived |
| `chore/*` | Dependency bumps, tooling, config | Short-lived |
| `docs/*` | Documentation only | Short-lived |
| `refactor/*` | Refactoring, no feature or fix | Short-lived |
| `test/*` | Test additions or fixes | Short-lived |
| `ci/*` | CI/CD pipeline changes | Short-lived |
| `release/X.x` | Versioned release branch, cut from `main` | Active while version is supported |
 
### Branch naming
 
```
<type>/<short-kebab-case-description>
```
 
Examples:
 
```
feat/oauth-login
fix/null-on-startup
hotfix/auth-bypass
chore/bump-dependencies
docs/update-contributing
refactor/extract-token-validator
ci/add-commitlint
```
 
Keep descriptions short — 2 to 5 words. If the work relates to a ticket, reference it in the PR description and commit footer (`Refs: VEM-123` or `Closes: VEM-123`), not in the branch name.
 
### Day-to-day workflow
 
1. Pull the latest `main`:
```bash
   git checkout main && git pull
```
2. Create your branch:
```bash
   git checkout -b feat/short-description
```
3. Do your work. Commit freely — no format required on branch commits.
4. Push and open a PR against `main`.
5. Write a PR title that follows Conventional Commits (see below).
6. Get at least one approval and ensure all PR Gate checks pass.
7. Squash and merge into `main`.
8. Delete the branch.
---
 
## PR titles: Conventional Commits
 
PR titles must follow [Conventional Commits v1.0.0](https://www.conventionalcommits.org/en/v1.0.0/). This is the only place it is enforced — individual commits on your branch are free-form.
 
Because we squash merge, the PR title becomes the single commit on `main`. This is what drives the changelog and git history, so it must be accurate and well-formed.
 
**This is enforced by CI. A non-conforming PR title will block merge.**
 
### Format
 
```
<type>(<optional scope>): <description>
```
 
### Types
 
| Type | When to use |
|---|---|
| `feat` | A new feature |
| `fix` | A bug fix |
| `docs` | Documentation only |
| `style` | Formatting or whitespace, no logic change |
| `refactor` | Code change that is neither a fix nor a feature |
| `perf` | Performance improvement |
| `test` | Adding or fixing tests |
| `build` | Build system or external dependencies |
| `ci` | CI/CD pipeline changes |
| `chore` | Other changes that don't modify src or test files |
| `revert` | Reverts a previous commit |
 
### Scope
 
Optional. A short identifier for the area of the codebase touched.
 
```
feat(auth): ...
fix(api): ...
chore(deps): ...
```
 
### Breaking changes
 
Append `!` after the type or scope, and include a `BREAKING CHANGE:` footer:
 
```
feat(api)!: rename /users endpoint to /accounts
 
BREAKING CHANGE: clients calling /users must update to /accounts
```
 
### Examples
 
```
feat(auth): add OAuth2 device flow
fix(api): handle null response from VEM telemetry endpoint
chore(deps): bump Newtonsoft.Json to 13.0.3
docs: clarify branch naming in CONTRIBUTING
refactor(parser): extract token validation into separate class
ci: add commitlint PR gate
feat(device)!: redesign handshake protocol
```
 
### Rules of thumb
 
- Use the imperative mood: "add" not "added" or "adds"
- No period at the end
- Keep it under 72 characters
- Reference tickets in the PR description or footer: `Refs: VEM-123` / `Closes: VEM-123`
---
 
## Pull requests
 
### Before opening
 
- Rebase or merge `main` into your branch so it is up to date
- Run tests locally
- Check your PR title follows Conventional Commits
- 
### PR description
 
Include:
- What changed and why
- Linked ticket if applicable (`Closes VEM-123`)
- Any deployment or migration notes
- Screenshots for UI changes

### Review and merge
 
- Minimum **1 approval** required for PRs into `main`
- Minimum **2 approvals** required for PRs into `release/*`
- All PR Gate status checks must pass
- All review conversations must be resolved
- **Squash and merge** — keeps `main` history linear and clean
- Delete the branch after merging
---
 
## Release branches
 
When cutting a versioned release:
 
1. Branch off `main`:
```bash
   git checkout -b release/1.x
```
2. Only bug fixes go on this branch — no new features
3. The release pipeline tags builds on this branch with the version number
4. Keep the branch alive for as long as the version is supported — archive it when EOL

### Fix-forward policy
 
Bugs are always fixed on `main` first, then cherry-picked to affected release branches:
 
1. Fix the bug on `main` via the normal PR workflow
2. Cherry-pick the resulting squash commit to the relevant `release/*` branch:
```bash
   git checkout release/1.x
   git cherry-pick <commit-sha>
```
3. Open a PR on the release branch for review
This ensures the fix is in all future work and cannot be accidentally missed in the next major version.
 
**Exception — emergency hotfix:** If production is down and the fix must ship immediately, branch off `release/*` directly, fix, and merge. Port the fix back to `main` within 24 hours. This is not the default — it is the emergency exception.
 
### Merging a release branch back into main
 
When merging `release/*` back into `main` (e.g. after a release stabilisation), use a **merge commit**, not squash. This preserves the release branch history.
 
```bash
git checkout main
git merge --no-ff release/1.x
```
 
This is the only case where a merge commit is used. Everything else is squash merge.
 
---
 
## PR Gate workflow
 
Every PR targeting `main` or `release/*` must pass the following checks before merge:
 
- Build validation (all target frameworks)
- Automated test execution
- Code coverage summary reporting
- Static analysis and linting
- **PR title validation (Conventional Commits)** — blocks merge if non-conforming
- Security scanning (Dependabot, CodeQL)
These are configured as required status checks in branch protection. They cannot be bypassed.
 
---
 
## Release workflow
 
Triggered automatically on merge to `main` or `release/*`:
 
- Package build
- Artifact versioning (`Major.Minor.BuildRunNumber`)
- Artifact storage
- Publish to target registry or release destination
- Changelog generation from PR titles
Deployment to production requires a **manual approval gate** in the pipeline.
 
---
 
## Branch protection
 
Applied to `main` and all `release/*` branches:
 
- No direct pushes — PRs only
- No force pushes
- No deletions
- Required reviews: 1 for `main`, 2 for `release/*`
- All required status checks must pass
- Applies to admins — no bypass
---
 
## What we don't do
 
- **No direct pushes to `main` or `release/*`.** Branch protection blocks this.
- **No force pushes to `main` or `release/*`.** Force-pushing your own feature branch before merging is fine.
- **No regular merge commits into `main`** except when merging a `release/*` branch back in.
- **No long-lived feature branches.** If a branch lives longer than a week, break the work into smaller PRs.
- **No new features on release branches.** Bug fixes only.
---
 
## Editor recommendations
 
These tools make Conventional Commits PR titles easier to write:
 
- JetBrains IDEs: "Conventional Commit" plugin
- CLI: [`commitizen`](https://github.com/commitizen/cz-cli) — `git cz` walks you through the format interactively
---
 
## Questions
 
Ask in the team channel, or open a draft PR and tag a reviewer for early feedback. Better to ask than to guess.
