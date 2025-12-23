---
description: Smart Git automation for Commits, Releases, and Hotfixes. Auto-updates README/CHANGELOG on release & enforces Conventional Commits.
---

# 🚀 Git Automation Workflow

This workflow automates the git lifecycle, ensuring conventional commits and proper release management.

## 1. Branch Safety Check
- Ensure you are on the correct branch (e.g., `main` for releases).
- Run `git pull --rebase` to ensure synchronization.

## 2. Modes

### 📝 Commit Mode (`/git commit`)
- **Action**: Stage and commit changes.
- **Rules**:
  - Enforce Conventional Commits (e.g., `feat:`, `fix:`, `docs:`, `chore:`, `refactor:`).
  - Automatically summarize changes if no message is provided.

### 📦 Release Mode (`/git release [major|minor|patch]`)
- **Action**: Prepare a new release.
- **Workflow**:
  1. **Update CHANGELOG.md**: Move content from `[Unreleased]` to a new version header with today's date.
  2. **Update README.md**: If versioning is mentioned in README, update it.
  3. **Stage & Commit**: `chore(release): prepare for version vX.X.X`.
  4. **Tag**: Create a signed/annotated git tag `vX.X.X`.
  5. **Push**: Execute `git push origin main --tags`.

### 🛡️ Hotfix Mode (`/git hotfix`)
- **Action**: Rapid fix for production issues.
- **Workflow**:
  1. Branch from last tag or main.
  2. Apply fix.
  3. Commit with `fix: ...`.
  4. Merge back and release patch.

## 3. Execution Logic for `push`
- If the command includes `push`, automatically run `git push origin [branch] --tags` after the primary action.

## ⚠️ Constraints
- NEVER force push unless explicitly asked.
- ALWAYS verify `git status` before starting.
- All commit messages must be in **English**.

// turbo-all
