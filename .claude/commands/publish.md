---
description: Security-scan, push to GitHub, refresh the README (with a screenshot of the live site) and repo About section, and deploy GitHub Pages
argument-hint: <owner/repo or GitHub URL> [branch]
allowed-tools: Read, Edit, Write, Glob, Grep, Bash(git status:*), Bash(git remote:*), Bash(git log:*), Bash(git diff:*), Bash(git ls-files:*), Bash(git branch:*), Bash(git add:*), Bash(git commit:*), Bash(git push:*), Bash(gh auth status:*), Bash(gh repo view:*), Bash(gh repo edit:*), Bash(gh api:*), Bash(gh run list:*), Bash(gh run watch:*), Bash(gh run view:*), mcp__playwright__browser_resize, mcp__playwright__browser_navigate, mcp__playwright__browser_take_screenshot, mcp__playwright__browser_console_messages, mcp__playwright__browser_close
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
- Live site links: v2, the current `index.html`, at `https://OWNER.github.io/REPO/v2/`, and v1 at `https://OWNER.github.io/REPO/` (confirm the real base URL in step 5 and correct it if it differs)
- The screenshot image line, if `docs/screenshot.png` already exists (step 6 adds it on a first publish and refreshes the image)
- What is in the page (sections and features), taken from the actual `index.html`
- How to run it locally
- Project structure and the single-file constraint
- How deployment works (the Pages workflow)
- A note that contact details and testimonials are placeholders

Base every statement on the files in the repo. Do not invent features, licences or badges.

## Step 3: create or update the GitHub Pages workflow

Check `.github/workflows/pages.yml`.

- If it exists, verify it is still correct: triggers on pushes to the published branch plus `workflow_dispatch`, has `pages: write` and `id-token: write` permissions, and stages only the site files, not `README.md`, `CLAUDE.md` or `.claude/`: the v1 page from `V1_COMMIT` at the site root, and the current `index.html` at `v2/` with the Content-Security-Policy injected. Never change `V1_COMMIT` or publish the current `index.html` at the root unless the user asks to replace v1. Edit only what is wrong.
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

## Step 6: capture a screenshot of the live site for the README

Use the Playwright MCP tools (the `playwright` server in `.mcp.json`). Take the screenshot from the live URL confirmed in step 5, after the deployment has succeeded, so it shows what is actually published. The Playwright server blocks `file:` URLs, so the local `index.html` cannot be used.

1. `browser_resize` to 1280 x 800.
2. `browser_navigate` to the v2 URL (the Pages URL followed by `v2/`).
3. `browser_take_screenshot` with `filename: docs/screenshot.png` (viewport only, not `fullPage`: content below the fold is hidden until it scrolls into view).
4. Read `docs/screenshot.png` and check it: the fonts have loaded and the hero statistics show their final values (15+, 1,200+, $500M) rather than a mid-count number. Also check `browser_console_messages` for Content-Security-Policy errors, which mean the deployed page is broken. If not, take it again.
5. `browser_close`.

Make sure the README shows the image directly under the live site link, adding the line if it is missing:

```markdown
![Screenshot of the Horizon Wealth Planning home page](docs/screenshot.png)
```

If `git status --short docs/screenshot.png README.md` shows a change, stage those two files by name, commit ("Refresh README screenshot") and push. This push redeploys the site; the page is unchanged, so do not take another screenshot. `.playwright-mcp/` holds Playwright's working files and is ignored by git; never stage it.

If the Playwright tools are not available, skip this step, leave any existing screenshot in place, and say so in the final report.

## Step 7: update the repo About section

Read the current values first so nothing useful is overwritten:

```bash
gh repo view OWNER/REPO --json description,homepageUrl,repositoryTopics
```

Then set the description (one sentence, under 350 characters), the website (the v2 URL) and a few accurate topics:

```bash
gh repo edit OWNER/REPO --description "<description>" --homepage "<pages url>" --add-topic <topic>
```

## Final report

Finish with a short summary:

- Security scan: what was checked and the result
- Commit hash and branch pushed
- Workflow run result with its link
- Live site URL
- Screenshot: refreshed, unchanged or skipped
- About section values now set
- Anything skipped or left for the user to do by hand, and why
