---
title: Git Workflow
description: Git branching strategy, commit conventions, and PR process for Cetmix Tower
category: development-workflows
order: 4
---

# Git Workflow

Git branching strategy, commit conventions, and pull request process for Cetmix Tower development.

## Branching Strategy

### Main Branches

- **`17.0`** - Main development branch (Odoo 17)
- **Release branches** - For stable releases (if used)

### Branch Naming

```
<type>/<short-description>

Examples:
feature/add-docker-integration
fix/server-connection-timeout
refactor/command-execution
docs/api-reference-update
```

**Types**:
- `feature/` - New features
- `fix/` - Bug fixes
- `refactor/` - Code refactoring
- `docs/` - Documentation updates
- `test/` - Test additions/updates
- `chore/` - Maintenance tasks

### Workflow

```
1. Fork repository
2. Create branch from 17.0
3. Make changes
4. Push to your fork
5. Create Pull Request to 17.0
6. Address review comments
7. Merge after approval
```

## Commit Message Conventions

Follow [Conventional Commits](https://www.conventionalcommits.org/) specification.

### Format

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### Types

- `feat`: New feature
- `fix`: Bug fix
- `refactor`: Code refactoring
- `docs`: Documentation changes
- `test`: Test additions/updates
- `style`: Code style changes (formatting, etc.)
- `chore`: Maintenance tasks
- `perf`: Performance improvements

### Examples

```bash
# Feature
git commit -m "feat: add Docker container management"

# Bug fix
git commit -m "fix: resolve SSH connection timeout issue"

# With scope
git commit -m "feat(webhook): add HMAC authentication support"

# With body
git commit -m "fix: prevent duplicate command execution

Check if command is already running before starting new execution.
Adds 'allow_parallel_run' field to control this behavior."

# Breaking change
git commit -m "feat!: change API authentication method

BREAKING CHANGE: API now requires Bearer token authentication.
Update your integration to include Authorization header."
```

### OCA Style (Alternative)

Cetmix Tower also accepts OCA-style commits:

```
[TAG] module: description

Examples:
[ADD] cetmix_tower_docker: Docker integration
[FIX] cetmix_tower_server: SSH timeout
[IMP] cetmix_tower_webhook: HMAC auth
[REF] cetmix_tower_server: Command execution
[REM] cetmix_tower_old: deprecated code
```

## Pull Request Process

### 1. Prepare Your PR

```bash
# Update your fork
git fetch upstream
git checkout 17.0
git merge upstream/17.0

# Rebase your branch
git checkout feature/my-feature
git rebase 17.0

# Run tests and linters
pre-commit run --all-files
odoo-bin --test-enable --test-tags cetmix_tower_server

# Push to your fork
git push origin feature/my-feature
```

### 2. Create Pull Request

**Title**: Follow commit message convention
```
feat: add Docker integration
fix: resolve SSH timeout issue
```

**Description Template**:
```markdown
## Summary
Brief description of changes

## Changes
- Added Docker client integration
- New model `cx.tower.docker.container`
- Docker commands support
- Documentation updates

## Test Plan
- [ ] Unit tests added and passing
- [ ] Integration tests passing
- [ ] Manual testing completed
- [ ] Documentation updated

## Breaking Changes
None / List breaking changes

## Screenshots (if applicable)
```

### 3. Code Review

**As Author**:
- Respond to comments promptly
- Make requested changes
- Push updates to same branch
- Mark conversations as resolved when addressed

**As Reviewer**:
- Review within 48 hours
- Be constructive and specific
- Test changes locally if significant
- Approve or request changes

### 4. Merge

- **Squash and merge** - For feature branches
- **Merge commit** - For multiple related commits
- **Rebase and merge** - For clean history

## Code Review Guidelines

### What to Review

✅ **Functionality**
- Does it work as intended?
- Are edge cases handled?
- Any obvious bugs?

✅ **Code Quality**
- Follows coding standards?
- Well-structured and readable?
- Appropriate naming?

✅ **Tests**
- Tests included for new code?
- Tests actually test the functionality?
- Good test coverage?

✅ **Documentation**
- Public methods documented?
- README updated if needed?
- Changelog updated?

✅ **Security**
- No security vulnerabilities?
- Proper input validation?
- Access rights checked?

✅ **Performance**
- No obvious performance issues?
- Efficient database queries?
- Appropriate indexing?

### Review Comments

```markdown
# Good comments

## Suggestion
Consider using `search_count()` instead of `len(search())` for better performance.

## Question
Should this raise `UserError` instead of `ValidationError`?

## Nitpick
Minor: Could we rename this variable to be more descriptive?

## Blocker
This will cause SQL injection. Use parameterized queries.

# Bad comments

❌ "This is wrong" (not specific)
❌ "Why did you do it this way?" (not constructive)
❌ "I would have done X" (without explaining why)
```

## Release Management

### Version Numbering

Follow [Semantic Versioning](https://semver.org/):
```
MAJOR.MINOR.PATCH

17.0.1.0.0
│  │ │ │ │
│  │ │ │ └─ Hotfix
│  │ │ └─── Patch
│  │ └───── Minor
│  └─────── Odoo version
└────────── Major (always matches Odoo)
```

### Creating a Release

```bash
# Update version in __manifest__.py
{
    "version": "17.0.2.0.0",
    # ...
}

# Update CHANGELOG
## [17.0.2.0.0] - 2024-01-15
### Added
- Docker integration
### Fixed
- SSH timeout issue

# Commit
git commit -m "chore: bump version to 17.0.2.0.0"

# Tag
git tag -a 17.0.2.0.0 -m "Release 17.0.2.0.0"

# Push
git push origin 17.0
git push origin 17.0.2.0.0
```

### Changelog Format

```markdown
# Changelog

## [Unreleased]
### Added
- New features not yet released

### Changed
- Changes in existing functionality

### Deprecated
- Features that will be removed

### Removed
- Removed features

### Fixed
- Bug fixes

### Security
- Security fixes

## [17.0.2.0.0] - 2024-01-15
### Added
- Docker container management
- HMAC authentication for webhooks

### Fixed
- SSH connection timeout after 30 seconds
- Variable rendering in Python commands
```

## Git Best Practices

### Commit Frequency

```bash
# Good - Logical units
git commit -m "feat: add server model"
git commit -m "feat: add server views"
git commit -m "test: add server tests"

# Bad - Too granular
git commit -m "add field"
git commit -m "fix typo"
git commit -m "fix another typo"
git commit -m "add another field"
```

### Atomic Commits

Each commit should:
- Represent one logical change
- Pass all tests
- Be deployable (for critical fixes)

### Branch Hygiene

```bash
# Keep branches up to date
git fetch upstream
git rebase upstream/17.0

# Clean up merged branches
git branch -d feature/merged-feature
git push origin --delete feature/merged-feature

# Interactive rebase to clean history
git rebase -i HEAD~5  # Last 5 commits
```

### Handling Merge Conflicts

```bash
# Update your branch
git fetch upstream
git rebase upstream/17.0

# If conflicts occur
# 1. Fix conflicts in files
# 2. Stage resolved files
git add resolved_file.py

# 3. Continue rebase
git rebase --continue

# Or abort if needed
git rebase --abort
```

## Common Scenarios

### Fix a Bug

```bash
# Create branch
git checkout -b fix/server-timeout 17.0

# Make changes
# Edit files...

# Commit
git commit -m "fix: resolve server connection timeout

Increase default timeout from 30 to 60 seconds.
Add configurable timeout parameter."

# Push and create PR
git push origin fix/server-timeout
```

### Add a Feature

```bash
# Create branch
git checkout -b feature/docker-support 17.0

# Make changes over multiple commits
git commit -m "feat: add docker client integration"
git commit -m "feat: add docker container model"
git commit -m "feat: add docker command support"
git commit -m "docs: add docker integration guide"
git commit -m "test: add docker integration tests"

# Push and create PR
git push origin feature/docker-support
```

### Update Documentation

```bash
# Create branch
git checkout -b docs/api-reference 17.0

# Make changes
git commit -m "docs: add webhook API reference"
git commit -m "docs: update installation guide"

# Push and create PR
git push origin docs/api-reference
```

## Troubleshooting

### Forgot to Create Branch

```bash
# Create branch from current state
git checkout -b feature/my-feature

# Your changes are now on the new branch
```

### Committed to Wrong Branch

```bash
# If not yet pushed
git reset --soft HEAD~1  # Undo commit, keep changes
git stash  # Stash changes
git checkout correct-branch
git stash pop  # Apply changes
git commit -m "correct commit message"
```

### Need to Update PR

```bash
# Make changes
# Edit files...

# Commit
git commit -m "fix: address review comments"

# Push (updates PR automatically)
git push origin feature/my-feature

# Or amend last commit
git commit --amend
git push origin feature/my-feature --force-with-lease
```

## Related Documentation

- [Coding Standards](01-coding-standards.md)
- [Testing Guidelines](02-testing-guidelines.md)
- [Development Workflows](README.md)
