---
description: Security-scan, push to GitHub, refresh the README and repo About section, and deploy GitHub Pages
argument-hint: <owner/repo or GitHub URL> [branch]
allowed-tools: Read, Edit, Write, Glob, Grep, Bash(git status:*), Bash(git remote:*), Bash(git log:*), Bash(git diff:*), Bash(git ls-files:*), Bash(git branch:*), Bash(git add:*), Bash(git commit:*), Bash(git push:*), Bash(gh auth status:*), Bash(gh repo view:*), Bash(gh repo edit:*), Bash(gh api:*), Bash(gh run list:*), Bash(gh run watch:*), Bash(gh run view:*)
---

# Publish this project to GitHub

Repo info from the user: `$ARGUMENTS`

## Current state

- Remotes: !`git remote -v`
- Branch: !`git branch --show-current`
- Status: !`git status --short`

## Step 0: resolve the target repo

Parse `$ARGUMENTS` into `OWNER/REPO` (accept `owner/repo`, an HTTPS URL or an SSH URL) and an optional branch (default: the current branch).

- If `$ARGUMENTS` is empty, use the existing `origin` remote. If there is no `origin` either, ask the user for the repo and stop until they answer.
- If `origin` exists but points to a different repo than the one given, ask the user before changing it. Never overwrite a remote silently.
- If there is no `origin`, add it: `git remote add origin https://github.com/OWNER/REPO.git`.
- Check `gh auth status`. If `gh` is missing or not signed in, say so, continue with the steps that only need `git`, and list the `gh` steps the user must do by hand at the end.

## Step 1: security scan (must pass before anything is pushed)

Scan everything that would be uploaded: tracked files (`git ls-files`), untracked files that are not ignored, and commits not yet on the remote (`git log origin/<branch>..HEAD -p`, or the whole history if the branch has never been pushed).

Look for:

- Secrets: API keys, tokens, passwords, private keys, connection strings. Patterns worth grepping for include `AKIA[0-9A-Z]{16}`, `gh[pousr]_[A-Za-z0-9]{36,}`, `github_pat_`, `sk-[A-Za-z0-9_-]{20,}`, `xox[baprs]-`, `AIza[0-9A-Za-z_-]{35}`, `-----BEGIN [A-Z ]*PRIVATE KEY-----`, and assignments such as `(password|passwd|secret|token|api[_-]?key)\s*[:=]\s*['"][^'"]{6,}`.
- Sensitive files: `.env*`, `*.pem`, `*.key`, `*.pfx`, `*.p12`, `id_rsa*`, `credentials*`, `*.sqlite`, `*.db`, `.claude/settings.local.json`, backups and dumps.
- Personal or internal data: real email addresses, phone numbers, street addresses, local user paths (`C:\Users\<name>`), internal hostnames or IPs, employee IDs.
- Whether the repo is public or private (`gh repo view OWNER/REPO --json visibility`), since that sets how much the findings matter.

The contact details in `index.html` are documented placeholders (see CLAUDE.md); report them as placeholders, not as findings.

Report the result as a short table: file, line, what was found, severity.

**If anything real is found, stop.** Do not push. Show the findings and propose a fix (remove the value, add the file to `.gitignore`, and note that a secret already in a commit must be rotated and removed from history). Continue only after the user confirms.

## Step 2: create or update the README

Read the existing `README.md` first and keep anything the user wrote by hand. It should cover:

- Project name and a one-paragraph description
- Live site link (`https://OWNER.github.io/REPO/`; confirm the real URL in step 5 and correct it if it differs)
- What is in the page (sections and features), taken from the actual `index.html`
- How to run it locally
- Project structure and the single-file constraint
- How deployment works (the Pages workflow)
- A note that contact details and testimonials are placeholders

Base every statement on the files in the repo. Do not invent features, licences or badges.

## Step 3: create or update the GitHub Pages workflow

Check `.github/workflows/pages.yml`.

- If it exists, verify it is still correct: triggers on pushes to the published branch plus `workflow_dispatch`, has `pages: write` and `id-token: write` permissions, and stages only the site files (`index.html` and any assets it references), not `README.md`, `CLAUDE.md` or `.claude/`. Edit only what is wrong.
- If it is missing, create it using `actions/checkout`, `actions/configure-pages`, `actions/upload-pages-artifact` and `actions/deploy-pages`.

Then make sure Pages is set to build from GitHub Actions:

```bash
gh api repos/OWNER/REPO/pages
```

If that returns 404, enable it with `gh api -X POST repos/OWNER/REPO/pages -f build_type=workflow`. If it exists with a different `build_type`, switch it with `-X PUT` and the same field.

## Step 4: commit and push

- Stage the specific files changed in steps 1 to 3 by name. Do not use `git add -A` or `git add .`.
- Show `git status --short` and `git diff --cached --stat`, then commit with a message describing what changed.
- Push with `git push -u origin <branch>`.
- Never force-push. If the push is rejected, stop and report why.

## Step 5: confirm the deployment

- Watch the workflow run triggered by the push: `gh run list --workflow pages.yml --limit 1`, then `gh run watch <id> --exit-status`.
- If it fails, read the log with `gh run view <id> --log-failed`, fix the cause, and push again.
- Get the real site URL: `gh api repos/OWNER/REPO/pages --jq .html_url`.
- If the URL differs from the one written in the README, correct the README, commit and push.

## Step 6: update the repo About section

Read the current values first so nothing useful is overwritten:

```bash
gh repo view OWNER/REPO --json description,homepageUrl,repositoryTopics
```

Then set the description (one sentence, under 350 characters), the website (the Pages URL from step 5) and a few accurate topics:

```bash
gh repo edit OWNER/REPO --description "<description>" --homepage "<pages url>" --add-topic <topic>
```

## Final report

Finish with a short summary:

- Security scan: what was checked and the result
- Commit hash and branch pushed
- Workflow run result with its link
- Live site URL
- About section values now set
- Anything skipped or left for the user to do by hand, and why
