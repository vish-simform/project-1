# Git & GitHub Workflow Lab

> A hands-on DevOps lab for **Vish**, **Smit**, and **Prat** — simulating real-world Git discipline without needing root/sudo or a specific tech stack.

---

## Table of Contents

1. [Overview](#1-overview)
2. [Roles & Rotation](#2-roles--rotation)
3. [Branching Strategy](#3-branching-strategy)
4. [Repository Structure](#4-repository-structure)
5. [Tooling & Prerequisites](#5-tooling--prerequisites)
6. [Constraint Index](#6-constraint-index)
7. [Lab Tasks — Phase by Phase](#7-lab-tasks--phase-by-phase)
   - [Phase 0: Repo Bootstrap (Vish)](#phase-0-repo-bootstrap-vish)
   - [Phase 1: GPG Signing Setup (All)](#phase-1-gpg-signing-setup-all)
   - [Phase 2: Pre-commit Hooks (All)](#phase-2-pre-commit-hooks-all)
   - [Phase 3: First Feature Cycle (Rotate)](#phase-3-first-feature-cycle-rotate)
   - [Phase 4: PR, Review & Merge (All)](#phase-4-pr-review--merge-all)
   - [Phase 5: Interactive Rebase & History Cleanup](#phase-5-interactive-rebase--history-cleanup)
   - [Phase 6: Conflict Resolution via Rebase](#phase-6-conflict-resolution-via-rebase)
   - [Phase 7: CODEOWNERS & Ownership Enforcement](#phase-7-codeowners--ownership-enforcement)
   - [Phase 8: Automated Release via Conventional Commits](#phase-8-automated-release-via-conventional-commits)
8. [Branch Protection Rules (GitHub Settings)](#8-branch-protection-rules-github-settings)
9. [Commit Message Cheatsheet](#9-commit-message-cheatsheet)
10. [Bonus Lab: GPG Identity Juggling (Shadow Workflow)](#10-bonus-lab-gpg-identity-juggling-shadow-workflow)

---

## 1. Overview

This lab enforces **9 professional Git constraints** in a safe, simulated monorepo. No application code is required — the repo contains plain text and shell scripts to simulate real change sets. All tooling runs on **Git + Python + standard Unix utilities** with no install privileges needed.

Each phase maps to one or more constraints. Roles rotate so every team member experiences every perspective — committer, reviewer, and gatekeeper.

---

## 2. Roles & Rotation

Roles rotate **per phase**. The table below shows the starting assignment. Shift left by one person each new phase.

| Role | Responsibility | Phase 1 | Phase 2 | Phase 3+ |
|---|---|---|---|---|
| **Repo Admin** | Branch protection, merge gatekeeper | Vish | Smit | Prat |
| **Feature Dev** | Writes commits, opens PRs | Smit | Prat | Vish |
| **Reviewer** | Reviews PR, enforces standards | Prat | Vish | Smit |

> **Rule:** No one merges their own PR. The Admin handles the final merge only after **both** other team members approve.

---

## 3. Branching Strategy

```
main
 └── dev
      └── <username>/feat/<short-description>
```

| Branch | Purpose | Who pushes |
|---|---|---|
| `main` | Production-stable, protected | No one directly — only via PR from `dev` |
| `dev` | Integration branch, protected | No one directly — only via PR from feature branches |
| `<user>/feat/<desc>` | Individual feature work | The assigned Feature Dev only, via PR |

**Branch naming examples:**
```
vish/feat/add-codeowners
smit/feat/setup-hooks
prat/feat/update-readme
```

**Lifecycle rule:** Feature branches are deleted immediately after their PR is merged (enforced via GitHub branch protection setting).

---

## 4. Repository Structure

```
/
├── README.md                  # This file (maintained by Repo Admin)
├── .github/
│   ├── workflows/
│   │   ├── ci.yml             # GitHub Actions — lint + verify signatures
│   │   └── release.yml        # GitHub Actions — automated changelog
│   └── CODEOWNERS             # Ownership rules
├── hooks/
│   └── pre-commit             # Shared pre-commit hook script
├── scripts/
│   └── simulate_change.py     # Helper to generate dummy file changes
├── vish/
│   └── notes.txt              # Vish's ownership zone
├── smit/
│   └── notes.txt              # Smit's ownership zone
├── prat/
│   └── notes.txt              # Prat's ownership zone
└── shared/
    └── config.txt             # Shared area — requires all owners to review
```

---

## 5. Tooling & Prerequisites

All tools below are available without install privileges.

| Tool | Used For | Check |
|---|---|---|
| `git` | Everything | `git --version` |
| `gpg` | Commit signing | `gpg --version` |
| `python3` | Hook scripts, change simulation | `python3 --version` |
| `bash` | Pre-commit hook | Available by default |
| GitHub Browser | PR creation, reviews, branch protection | Log in at github.com |

**No npm, no Node, no sudo required.**

---

## 6. Constraint Index

| # | Constraint | Enforced By |
|---|---|---|
| C1 | Commit signing with GPG verification | `git config`, GitHub branch protection |
| C2 | Linear history — no merge commits | `git rebase`, branch protection (require linear) |
| C3 | Atomic commits — single responsibility | Code review + PR feedback |
| C4 | Conventional commit messages | Pre-commit hook (regex check) |
| C5 | Pre-commit hooks — static checks | `hooks/pre-commit` shell script |
| C6 | No direct push — PRs only | Branch protection on `main` and `dev` |
| C7 | Monorepo with CODEOWNERS enforcement | `.github/CODEOWNERS` + branch protection |
| C8 | Interactive rebase before merge | Team process — enforced in PR checklist |
| C9 | Branch naming + auto-delete on merge | Branch protection settings |

---

## 7. Lab Tasks — Phase by Phase

---

### Phase 0: Repo Bootstrap (Vish)

**Who:** Vish (Repo Admin)
**Constraints practiced:** C6, C9

#### Steps

**0.1 — Initialize repo locally**
```bash
git init git-workflow-lab
cd git-workflow-lab
git checkout -b main
```

**0.2 — Create the skeleton structure**
```bash
mkdir -p .github/workflows hooks scripts vish smit prat shared

echo "# Git & GitHub Workflow Lab" > README.md
echo "Vish's notes." > vish/notes.txt
echo "Smit's notes." > smit/notes.txt
echo "Prat's notes." > prat/notes.txt
echo "shared config v1" > shared/config.txt
echo "# pre-commit hook placeholder" > hooks/pre-commit
chmod +x hooks/pre-commit
```

**0.3 — First commit on main (only exception to no-direct-push rule)**
```bash
git add .
git commit -S -m "chore: initial repo scaffold"
git push origin main
```

> This is the **only** direct push to `main` permitted. All future changes go through PRs.

**0.4 — Create and push `dev` branch**
```bash
git checkout -b dev
git push origin dev
```

**0.5 — Configure branch protection on GitHub (browser)**

Go to: `Settings → Branches → Add rule`

For **both** `main` and `dev`, enable:
- [x] Require a pull request before merging
- [x] Require approvals: **2**
- [x] Dismiss stale pull request approvals when new commits are pushed
- [x] Require status checks to pass before merging
- [x] Require branches to be up to date before merging
- [x] Require linear history
- [x] Require signed commits
- [x] Automatically delete head branches

**0.6 — Invite collaborators**

Go to: `Settings → Collaborators → Add people` → Add Smit and Prat with **Write** access.

---

### Phase 1: GPG Signing Setup (All)

**Who:** All three independently on their own machines
**Constraints practiced:** C1

Each person runs these steps individually.

**1.1 — Check existing GPG keys**
```bash
gpg --list-secret-keys --keyid-format=long
```

**1.2 — Generate a key if none exists**
```bash
gpg --full-generate-key
# Choose: RSA and RSA → 4096 bits → no expiry (or set one) → enter name and work email
```

**1.3 — Export your key ID**
```bash
gpg --list-secret-keys --keyid-format=long
# Note the key ID from the line: sec   rsa4096/<KEY_ID>
```

**1.4 — Configure Git to use your key**
```bash
git config --global user.signingkey <YOUR_KEY_ID>
git config --global commit.gpgsign true
git config --global tag.gpgsign true
```

**1.5 — Export public key and add to GitHub**
```bash
gpg --armor --export <YOUR_KEY_ID>
# Copy the full output
```
Go to: `GitHub → Settings → SSH and GPG keys → New GPG key` → paste and save.

**1.6 — Verify signing works**
```bash
git commit --allow-empty -S -m "test: verify gpg signing"
git log --show-signature -1
# Should show: "Good signature from..."
```

> **Expected:** GitHub shows a green **Verified** badge on all your commits going forward.

---

### Phase 2: Pre-commit Hooks (All)

**Who:** All three independently
**Constraints practiced:** C4, C5

Git hooks are local to each machine. Each person installs the shared hook from the repo.

**2.1 — Clone the repo (Smit and Prat)**
```bash
git clone git@github.com:<org>/git-workflow-lab.git
cd git-workflow-lab
```

**2.2 — Install the shared hook**
```bash
cp hooks/pre-commit .git/hooks/pre-commit
cp hooks/commit-msg .git/hooks/commit-msg
chmod +x .git/hooks/pre-commit .git/hooks/commit-msg
```

**2.3 — Write the hook (Vish opens a PR to add this to `hooks/pre-commit`)**

The hook enforces:
- Conventional commit message format
- No trailing whitespace in staged files
- No file larger than 1MB is committed

```bash
#!/usr/bin/env bash
# hooks/pre-commit — runs before every commit

set -e

# ── 1. Conventional commit format check ──────────────────────────────────────
COMMIT_MSG_FILE=".git/COMMIT_EDITMSG"

# If interactive, read from stdin. During hook, read from file.
if [ -f "$COMMIT_MSG_FILE" ]; then
  MSG=$(cat "$COMMIT_MSG_FILE")
else
  MSG=$(git log -1 --pretty=%B 2>/dev/null || echo "")
fi

# Pattern: type(optional-scope): description
PATTERN="^(feat|fix|chore|docs|style|refactor|test|ci|perf|build|revert)(\([a-z0-9-]+\))?: .{1,72}$"

# Only validate if we have a message (skip during --allow-empty tests)
if [ -n "$MSG" ]; then
  if ! echo "$MSG" | grep -Eq "$PATTERN"; then
    echo ""
    echo "❌ COMMIT BLOCKED — Invalid commit message format."
    echo ""
    echo "   Got:      $MSG"
    echo "   Expected: <type>(<scope>): <description>"
    echo "   Example:  feat(vish): add codeowners file"
    echo ""
    echo "   Valid types: feat fix chore docs style refactor test ci perf build revert"
    echo ""
    exit 1
  fi
fi

# ── 2. Trailing whitespace check ──────────────────────────────────────────────
STAGED=$(git diff --cached --name-only --diff-filter=ACM)

for FILE in $STAGED; do
  if grep -Pq " +$" "$FILE" 2>/dev/null; then
    echo "❌ COMMIT BLOCKED — Trailing whitespace found in: $FILE"
    echo "   Fix with: sed -i 's/[[:space:]]*$//' $FILE"
    exit 1
  fi
done

# ── 3. File size check (max 1MB) ──────────────────────────────────────────────
for FILE in $STAGED; do
  SIZE=$(wc -c < "$FILE")
  if [ "$SIZE" -gt 1048576 ]; then
    echo "❌ COMMIT BLOCKED — File too large (>1MB): $FILE ($SIZE bytes)"
    exit 1
  fi
done

echo "✅ Pre-commit checks passed."
exit 0
```

**2.4 — Test the hook**

```bash
# Should FAIL (bad message format)
git commit --allow-empty -m "updated stuff"

# Should PASS
git commit --allow-empty -S -m "chore: test pre-commit hook"
```

> **Note:** The commit message check inside hooks runs at `commit-msg` hook stage, but for simplicity this combined hook covers all checks at `pre-commit`. For production setups, split into `pre-commit` and `commit-msg`.

---

### Phase 3: First Feature Cycle (Rotate)

**Who:** Assigned Feature Dev (see rotation table in Section 2)
**Constraints practiced:** C2, C3, C4, C9

**3.1 — Sync with upstream `dev`**
```bash
git fetch origin
git checkout dev
git pull --rebase origin dev
```

**3.2 — Create your feature branch**
```bash
# Replace <username> and <description> appropriately
git checkout -b vish/feat/add-codeowners
```

**3.3 — Make an atomic change**

Use the simulate script or edit files directly:
```bash
python3 scripts/simulate_change.py --file vish/notes.txt --line "Added CODEOWNERS planning notes."
```

Or manually:
```bash
echo "CODEOWNERS planning: vish owns /vish, smit owns /smit, prat owns /prat" >> vish/notes.txt
```

**3.4 — Stage and commit (signed, conventional)**
```bash
git add vish/notes.txt
git commit -S -m "feat(vish): add codeowners planning notes"
```

> Each commit = one logical change. If you catch yourself writing "and" in a commit message, it's two commits.

**3.5 — Push your branch**
```bash
git push origin vish/feat/add-codeowners
```

> ⛔ **Never push directly to `dev` or `main`.** Branch protection will reject it, but don't attempt it.

---

### Phase 4: PR, Review & Merge (All)

**Who:** Feature Dev opens PR; both other teammates review; Admin merges
**Constraints practiced:** C2, C3, C6, C7

**4.1 — Open the PR (Feature Dev, via browser)**

- Base: `dev`
- Compare: your feature branch
- Title must follow conventional commit format: `feat(vish): add codeowners planning notes`
- Description must include:
  ```
  ## What changed
  <one sentence>

  ## Why
  <one sentence>

  ## Checklist
  - [ ] Commits are signed (Verified badge visible)
  - [ ] Each commit is atomic (single responsibility)
  - [ ] Commit messages follow conventional format
  - [ ] Branch is rebased on latest dev
  - [ ] Interactive rebase done — no WIP/fixup commits in history
  ```

**4.2 — Review the PR (Both reviewers)**

Each reviewer checks:
- All commits show **Verified** badge (GPG signed)
- No merge commits in the history (`git log --oneline` should be linear)
- Commit messages match the conventional format
- Changes are scoped correctly (no one edits outside their ownership zone without justification)
- No trailing whitespace, no oversized files

Leave inline comments for any issues. **Do not approve until all checklist items pass.**

**4.3 — Address review feedback (Feature Dev)**

```bash
# Make requested changes
echo "Updated per review feedback." >> vish/notes.txt
git add vish/notes.txt
git commit -S -m "fix(vish): address review feedback on notes"
git push origin vish/feat/add-codeowners
```

**4.4 — Merge (Repo Admin, via browser)**

Once both teammates have approved:
- Select **"Rebase and merge"** (never "Create a merge commit")
- Confirm the merge
- Delete the feature branch (should auto-delete per protection settings)

---

### Phase 5: Interactive Rebase & History Cleanup

**Who:** Feature Dev (before opening a PR)
**Constraints practiced:** C2, C8

This phase simulates a messy commit history that needs cleaning before review.

**5.1 — Create a messy branch**
```bash
git checkout dev && git pull --rebase origin dev
git checkout -b smit/feat/update-shared-config

echo "change 1" >> shared/config.txt
git add . && git commit -S -m "wip: first attempt"

echo "change 2" >> shared/config.txt
git add . && git commit -S -m "fix typo"

echo "change 3" >> shared/config.txt
git add . && git commit -S -m "feat(shared): final update to config"
```

**5.2 — Inspect the messy history**
```bash
git log --oneline
# You'll see 3 commits, two of which are noise
```

**5.3 — Interactive rebase to squash/rename**
```bash
git rebase -i origin/dev
```

In the editor that opens, change:
```
pick abc1234 wip: first attempt
pick def5678 fix typo
pick ghi9012 feat(shared): final update to config
```
To:
```
squash abc1234 wip: first attempt
squash def5678 fix typo
pick ghi9012 feat(shared): final update to config
```

Save and close. In the next editor, write the final clean commit message:
```
feat(shared): update shared config with v2 values
```

**5.4 — Verify clean history**
```bash
git log --oneline
# Should now show exactly ONE commit
```

**5.5 — Force-push the cleaned branch**
```bash
git push --force-with-lease origin smit/feat/update-shared-config
```

> `--force-with-lease` is safer than `--force` — it fails if someone else pushed to the branch since your last fetch.

---

### Phase 6: Conflict Resolution via Rebase

**Who:** Any two members simultaneously (simulate parallel work)
**Constraints practiced:** C2

**6.1 — Simulate parallel branches**

Both Smit and Prat (or any two members) create branches off the same `dev` state and both edit `shared/config.txt`.

```bash
# Member A
git checkout -b vish/feat/config-update-a
echo "config key A = value1" >> shared/config.txt
git add . && git commit -S -m "feat(shared): add config key A"
git push origin vish/feat/config-update-a
# Open PR immediately

# Member B (simultaneously)
git checkout dev
git checkout -b smit/feat/config-update-b
echo "config key B = value2" >> shared/config.txt
git add . && git commit -S -m "feat(shared): add config key B"
git push origin smit/feat/config-update-b
```

**6.2 — Merge A's PR first (Admin)**

Normal PR + review + rebase merge.

**6.3 — B must now rebase onto updated `dev`**
```bash
git fetch origin
git checkout smit/feat/config-update-b
git rebase origin/dev
# Git will stop at conflict — open shared/config.txt and resolve manually
```

Resolve the conflict, keeping both keys:
```
config key A = value1
config key B = value2
```

```bash
git add shared/config.txt
git rebase --continue
git push --force-with-lease origin smit/feat/config-update-b
```

**6.4 — Open PR for B and complete normal review cycle**

> ✅ Outcome: Both changes land in a straight linear history — no merge commits anywhere.

---

### Phase 7: CODEOWNERS & Ownership Enforcement

**Who:** Vish sets up; all members validate
**Constraints practiced:** C7

**7.1 — Create `.github/CODEOWNERS`**
```bash
git checkout dev && git pull --rebase origin dev
git checkout -b vish/feat/add-codeowners
```

Create `.github/CODEOWNERS`:
```
# Global fallback — Vish reviews everything not explicitly owned
*                   @vish-github-handle

# Per-person ownership zones
/vish/              @vish-github-handle
/smit/              @smit-github-handle
/prat/              @prat-github-handle

# Shared area — all three must review
/shared/            @vish-github-handle @smit-github-handle @prat-github-handle

# Workflow files — only admin can modify
/.github/           @vish-github-handle
```

> Replace `@vish-github-handle` etc. with actual GitHub usernames.

**7.2 — Commit and raise PR**
```bash
git add .github/CODEOWNERS
git commit -S -m "chore: add CODEOWNERS for ownership enforcement"
git push origin vish/feat/add-codeowners
```

**7.3 — Enable CODEOWNERS review requirement (browser)**

Go to: `Settings → Branches → Edit rule for main and dev`
- [x] Require review from Code Owners

**7.4 — Validate enforcement**

Smit creates a PR that edits `/prat/notes.txt`. GitHub should automatically request Prat's review — verify this happens.

---

### Phase 8: Automated Release via Conventional Commits

**Who:** Vish sets up GitHub Actions; all members trigger it
**Constraints practiced:** C4

**8.1 — Create `.github/workflows/release.yml`**
```yaml
name: Auto Release Notes

on:
  push:
    branches:
      - main

jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write

    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Generate Changelog
        id: changelog
        run: |
          echo "## What Changed" > CHANGELOG_DRAFT.md
          echo "" >> CHANGELOG_DRAFT.md

          echo "### Features" >> CHANGELOG_DRAFT.md
          git log $(git describe --tags --abbrev=0 2>/dev/null || git rev-list --max-parents=0 HEAD)..HEAD \
            --pretty=format:"- %s (%an)" | grep "^- feat" >> CHANGELOG_DRAFT.md || echo "_none_" >> CHANGELOG_DRAFT.md

          echo "" >> CHANGELOG_DRAFT.md
          echo "### Fixes" >> CHANGELOG_DRAFT.md
          git log $(git describe --tags --abbrev=0 2>/dev/null || git rev-list --max-parents=0 HEAD)..HEAD \
            --pretty=format:"- %s (%an)" | grep "^- fix" >> CHANGELOG_DRAFT.md || echo "_none_" >> CHANGELOG_DRAFT.md

          echo "" >> CHANGELOG_DRAFT.md
          echo "### Chores" >> CHANGELOG_DRAFT.md
          git log $(git describe --tags --abbrev=0 2>/dev/null || git rev-list --max-parents=0 HEAD)..HEAD \
            --pretty=format:"- %s (%an)" | grep "^- chore" >> CHANGELOG_DRAFT.md || echo "_none_" >> CHANGELOG_DRAFT.md

          cat CHANGELOG_DRAFT.md

      - name: Create Tag and Release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          VERSION="v$(date +'%Y%m%d%H%M%S')"
          git tag "$VERSION"
          git push origin "$VERSION"

          gh release create "$VERSION" \
            --title "Release $VERSION" \
            --notes-file CHANGELOG_DRAFT.md
```

**8.2 — Commit and push via PR**
```bash
git add .github/workflows/release.yml
git commit -S -m "ci: add automated release notes workflow"
```

**8.3 — Validate**

Once merged to `main`, go to `Actions` tab — the workflow should run and create a GitHub Release with a changelog grouped by `feat`, `fix`, and `chore`.

> This is the payoff for following conventional commits consistently — your release notes write themselves.

---

## 8. Branch Protection Rules (GitHub Settings)

Apply these settings to **both** `main` and `dev` branches.

| Setting | Value |
|---|---|
| Require pull request before merging | ✅ Enabled |
| Required approvals | **2** |
| Dismiss stale reviews on new push | ✅ Enabled |
| Require status checks to pass | ✅ Enabled |
| Require branches to be up to date | ✅ Enabled |
| Require linear history | ✅ Enabled |
| Require signed commits | ✅ Enabled |
| Require code owner reviews | ✅ Enabled |
| Automatically delete head branches | ✅ Enabled |
| Allow force pushes | ❌ Disabled |
| Allow deletions | ❌ Disabled |

---

## 9. Commit Message Cheatsheet

```
<type>(<scope>): <short description>

Types:
  feat      → new feature or addition
  fix       → bug fix or correction
  chore     → maintenance, tooling, config
  docs      → documentation only
  refactor  → restructure without behaviour change
  test      → add or update tests
  ci        → changes to CI/CD workflows
  style     → formatting, whitespace (no logic change)
  perf      → performance improvement
  revert    → reverts a previous commit

Scope (optional):
  Use the folder/area being changed: vish, smit, prat, shared, hooks, ci

Rules:
  - Max 72 characters on the first line
  - Lowercase type and scope
  - No period at the end
  - Use imperative mood: "add" not "added" or "adds"

Examples:
  feat(vish): add deployment checklist to notes
  fix(shared): correct config key naming
  chore: update pre-commit hook pattern
  ci: add signature verification step to workflow
  docs: update README with phase 6 instructions
```

---

## 10. Bonus Lab: GPG Identity Juggling (Shadow Workflow)

> **Run this in a separate throwaway repo.** Not connected to the main lab.
> **Purpose:** Learn how to manage multiple GPG identities — simulating a scenario where one public identity makes commits but a second person contributes under that identity.

---

### Why This Matters in DevOps

In real environments: shared service accounts, CI bots, pair programming on a single machine, or contractor work under a client identity all require juggling GPG keys safely.

---

### Setup

**B.1 — Person B generates a key with Person A's identity details (simulated)**
```bash
# On Person B's machine — create a key that mimics A's identity
gpg --batch --gen-key <<EOF
%no-protection
Key-Type: RSA
Key-Length: 4096
Name-Real: Vish (Shadow)
Name-Email: vish-work@example.com
Expire-Date: 0
EOF
```

**B.2 — Export and import between machines**
```bash
# On B's machine — export
gpg --armor --export-secret-keys vish-work@example.com > vish_shadow.asc

# On A's machine — import B's key
gpg --import vish_shadow.asc
gpg --list-secret-keys  # Both keys now visible
```

**B.3 — Switch active signing key per commit**
```bash
# Temporarily override signing key for one commit
git -c user.signingkey=<B_KEY_ID> commit -S -m "feat: shadow commit from B under A identity"

# Or set per-repo
git config user.signingkey <B_KEY_ID>
```

**B.4 — Best practices for key juggling**

| Practice | Why |
|---|---|
| Always use `--force-with-lease` when force-pushing | Prevents overwriting others' work |
| Keep a `~/.gnupg/` backup of all keys | Loss = unrecoverable signed history |
| Use `gpg --list-secret-keys --keyid-format=long` to confirm active key before commits | Prevents signing with the wrong identity |
| Revoke shadow keys immediately after use | Limits exposure window |
| Never store raw secret keys in the repo | Even in `.gitignore`d files — use external secret managers |

**B.5 — Clean up after the exercise**
```bash
# Delete the shadow key from your keyring
gpg --delete-secret-and-public-key <SHADOW_KEY_ID>
```

---

*Lab designed for DevOps/cloud trainees. No root access, no language runtimes beyond Git + Python + standard Unix tools required.*