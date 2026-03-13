# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A GitHub Action (forked from ClearTax/jira-lint) that lints PRs against JIRA. It extracts JIRA issue keys from branch names, fetches issue details via the JIRA REST API, then updates PR descriptions with issue metadata, adds labels, and posts comments about PR quality (title similarity, size).

## Build & Development Commands

- **Build:** `npm run build` (uses `@vercel/ncc` to bundle `src/main.ts` into `lib/index.js`)
- **Test:** `npm test` (Jest with ts-jest)
- **Test single file:** `npx jest __tests__/utils.test.ts`
- **Lint:** `npm run lint` (Prettier + ESLint)
- **Node version:** 22 for development (see `.nvmrc`); action runtime is `node20` in `action.yml` (GitHub Actions skips node22, upgrade to `node24` when available ~June 2026)

The `lib/` directory contains the compiled/bundled output and is checked into git (required for GitHub Actions). **You must run `npm run build` and commit `lib/` after any source change.** CI verifies `lib/` matches the build output and will fail if it's stale.

## Architecture

The action entry point is `src/main.ts` which:
1. Reads GitHub Action inputs (tokens, JIRA base URL, skip patterns, thresholds)
2. Extracts JIRA issue keys from the PR's head branch name using a reversed-string regex match (`src/utils.ts:getJIRAIssueKeys`)
3. Fetches issue details from JIRA REST API v3 via an axios client (`src/utils.ts:getJIRAClient`)
4. Updates the PR: adds labels (project name, hotfix status, issue type), prepends issue details to PR body, posts comments about title quality and PR size
5. Optionally validates issue status against allowed statuses

Key design details:
- **Branch name parsing** reverses the input string before regex matching (`JIRA_REGEX_MATCHER` in `constants.ts`), then reverses matches back. The last match is used as the issue key.
- **Idempotent PR updates:** A hidden HTML marker (`added_by_jira_lint`) in the PR body prevents duplicate description updates.
- **Two HTTP clients:** Octokit for GitHub API, axios for JIRA API. The `node-fetch` package is passed to Octokit's `request.fetch`.

Source files: `src/main.ts` (entry), `src/utils.ts` (all logic), `src/types.ts` (TypeScript types), `src/constants.ts` (regex patterns, defaults).
