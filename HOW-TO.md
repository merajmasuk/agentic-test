# HOW-TO: AI-Assisted GitHub Workflow

This document walks through how this repository's initial setup and test
run were carried out using Claude, step by step, as a reference for
repeating or extending the process.

## Overview

The workflow lets Claude design, write, commit, and push code changes via
a dedicated branch and pull request, without needing standing access to
your GitHub account. A short-lived Personal Access Token (PAT) is
supplied per session and never persisted.

## Step-by-step process

1. **Design phase (chat only).**
   Requirements and file structure were discussed in chat before any
   files were written — no tool calls, no disk writes.

2. **Repo creation.**
   Since GitHub web login was temporarily unavailable, the repository
   (`agentic-test`, private) was created manually via the GitHub Android
   app instead of by Claude directly.

3. **Generate a PAT.**
   A Personal Access Token was generated from GitHub's web settings
   (Settings → Developer settings → Personal access tokens), scoped to
   this repository with:
   - `Contents: Read and write`
   - `Pull requests: Read and write`

   Fine-grained tokens had a UI bug where repo selection didn't save
   reliably, so a classic token (`repo` scope) was used as a fallback
   for the first test.

4. **Token handed to Claude.**
   The token was pasted directly into chat. Claude used it only for
   that session's git/API calls and never wrote it to a persisted file.

5. **Clone and branch.**
   Claude cloned the repository into its sandbox and created a new
   branch dedicated to the change (e.g. `ai/test-run`, `ai/add-howto`)
   — never committing directly to `main`.

6. **Single write pass.**
   All files for the change were created in one continuous step, right
   after the design phase concluded, to avoid a sandbox reset splitting
   the work.

7. **Commit with clear identity.**
   `git config user.name` / `user.email` were set to the user's own
   identity so commits are attributed correctly, with a
   `Co-authored-by: Claude <noreply@anthropic.com>` trailer added for
   transparency.

8. **Push and open a PR.**
   The branch was pushed to GitHub, then a pull request was opened via
   the GitHub REST API from the feature branch into `main`, with a
   description summarizing the change.

9. **Human review and merge.**
   Claude never merges. The user reviewed the PR on GitHub and merged
   it manually.

10. **Updating after a merge.**
    Because Claude's sandbox does not persist between sessions, GitHub
    itself is treated as the source of truth. Before starting a new
    change, Claude re-clones the repository fresh (picking up any
    merged changes) rather than assuming local files are current.

11. **Token cleanup.**
    Since the token was shared in a chat transcript, it was revoked
    immediately after each session, and a new one generated for the
    next round of changes.

## Notes and gotchas encountered

- An empty repository has no default branch history, so the very first
  push became the de facto default branch. This was fixed by explicitly
  creating an empty `main` base commit, setting it as the default
  branch, and rebasing the feature branch onto it before opening a PR.
- GitHub's fine-grained PAT repository-selection UI can silently fail
  to save the selected repo — if a token returns 404 on a repo it
  should have access to, try regenerating, or fall back to a classic
  token scoped to `repo`.
- Any token pasted into a chat session should be treated as compromised
  once the session ends, regardless of its configured expiry, and
  revoked proactively.
