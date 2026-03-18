# jira-lint

> A light-weight lint workflow when using GitHub along with [JIRA][jira] for project management.
> Forked from [ClearTax/jira-lint](https://github.com/ClearTax/jira-lint).

---

<!-- toc -->

- [Installation](#installation)
- [Features](#features)
  - [PR Status Checks](#pr-status-checks)
  - [PR Description & Labels](#pr-description--labels)
    - [Description](#description)
    - [Labels](#labels)
    - [PR Title Validation](#pr-title-validation)
    - [Issue Status Validation](#issue-status-validation)
    - [Soft-validations via comments](#soft-validations-via-comments)
  - [Options](#options)
  - [`jira-token`](#jira-token)
  - [Skipping branches](#skipping-branches)
- [Contributing](#contributing)
- [FAQ](#faq)

<!-- tocstop -->

## Installation

To make `jira-lint` a part of your workflow, just add a `jira-lint.yml` file in your `.github/workflows/` directory in your GitHub repository.

```yml
name: jira-lint
on: [pull_request]

jobs:
  Jira:
    runs-on: ubuntu-latest
    steps:
      - uses: CollabIP/jira-lint@master
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          jira-token: ${{ secrets.JIRA_TOKEN }}
          jira-base-url: ${{ vars.JIRA_ENDPOINT }}
          skip-comments: false
          skip-branches: '^master(-\w*)?'
```

It can also be used as part of an existing workflow by adding it as a step. More information about the [options here](#options).

## Features

### PR Status Checks

`jira-lint` adds a status check which helps you avoid merging PRs which are missing a valid Jira Issue Key in the branch name. It will use the [Jira API](https://developer.atlassian.com/cloud/jira/platform/rest/v3/) to validate a given key.

### PR Description & Labels

#### Description

When a PR passes the above check, `jira-lint` will also add the issue details to the top of the PR description. It will pick details such as the Issue summary, type, estimation points, status and labels and add them to the PR description.

#### Labels

`jira-lint` will automatically label PRs with:

- A label based on the Jira Project name (the project the issue belongs to). For example, if your project name is `Escher` then it will add `escher` as a label.
- `HOTFIX-PROD` - if the PR is raised against `production-release`.
- `HOTFIX-PRE-PROD` - if the PR is raised against `release/v*`.
- Jira issue type ([based on your project](https://confluence.atlassian.com/adminjiracloud/issue-types-844500742.html)).

#### PR Title Validation

By default, `jira-lint` requires that PR titles start with the JIRA issue key extracted from the branch name (e.g. `ABC-123: Fix the thing`). This ensures PR titles are easily traceable back to Jira issues. Set `validate_pr_title` to `false` to disable this check.

#### Issue Status Validation

Issue status is shown in the [Description](#description).

**Why validate issue status?**
In some cases, one may be pushing changes for a story that is set to `Done`/`Completed` or it may not have been pulled into working backlog or current sprint.

This option allows discouraging pushing to branches for stories that are set to statuses other than the ones allowed in the project; for example - you may want to only allow PRs for stories that are in `To Do`/`Planning`/`In Progress` states.

The following flags can be used to validate issue status:
- `validate_issue_status` - If set to `true`, `jira-lint` will validate the issue status based on `allowed_issue_statuses`.
- `allowed_issue_statuses` - A comma separated list of statuses. Only used when `validate_issue_status` is `true`. If the detected issue's status is not in the list, the status check will fail.

#### Soft-validations via comments

`jira-lint` will add comments to a PR to encourage better PR practices (unless `skip-comments` is `true`):

- **PR title similarity** - comments on how well the PR title matches the Jira issue summary.
- **PR size** - adds a comment discouraging PRs that exceed the `pr-threshold` line count.

### Options

| key                      | description                                                                                                                                                                                                     | required | default       |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ------------- |
| `github-token`           | Token used to update PR description and add labels. `GITHUB_TOKEN` is already available [when you use GitHub actions](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/authenticating-with-the-github_token#about-the-github_token-secret), so all that is required is to pass it as a param here. | true     | null          |
| `jira-token`             | Token used to fetch Jira Issue information. Check [below](#jira-token) for more details on how to generate the token. | true     | null          |
| `jira-base-url`          | The subdomain of JIRA cloud that you use to access it. Ex: `https://your-domain.atlassian.net`. | true     | null          |
| `skip-branches`          | A regex to ignore running `jira-lint` on certain branches, like production etc. | false    | `''`          |
| `skip-comments`          | A `Boolean` if set to `true` then `jira-lint` will skip adding lint comments for PR title and size. | false    | `false`       |
| `pr-threshold`           | An `Integer` based on which `jira-lint` will add a comment discouraging huge PRs. Disabled by `skip-comments`. | false    | `800`         |
| `validate_pr_title`      | A `Boolean` if set to `true`, requires PR titles to start with the JIRA issue key from the branch name (e.g. `ABC-123: ...`). | false    | `true`        |
| `validate_issue_status`  | A `Boolean` if set to `true`, validates the status of the detected Jira issue against `allowed_issue_statuses`. | false    | `false`       |
| `allowed_issue_statuses` | A comma separated list of allowed statuses. Requires `validate_issue_status` to be `true`. | false    | `"In Progress"` |

### `jira-token`

Since tokens are private, we suggest adding them as [GitHub secrets](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/creating-and-using-encrypted-secrets).

The Jira token is used to fetch issue information via the Jira REST API. To get the token:
1. Generate an [API token via JIRA](https://confluence.atlassian.com/cloud/api-tokens-938839638.html).
2. Create the encoded token in the format of `base64Encode(<username>:<api_token>)`.
   For example, if the username is `ci@example.com` and the token is `954c38744be9407ab6fb`, then `ci@example.com:954c38744be9407ab6fb` needs to be base64 encoded to form `Y2lAZXhhbXBsZS5jb206OTU0YzM4NzQ0YmU5NDA3YWI2ZmI=`.
3. The above value needs to be added as the `JIRA_TOKEN` secret in your GitHub project.

Note: The user should have the [required permissions (mentioned under GET Issue)](https://developer.atlassian.com/cloud/jira/platform/rest/v3/?utm_source=%2Fcloud%2Fjira%2Fplatform%2Frest%2F&utm_medium=302#api-rest-api-3-issue-issueIdOrKey-get).

### Skipping branches

Since GitHub actions take string inputs, `skip-branches` must be a regex which will work for all sets of branches you want to ignore. This is useful for merging protected/default branches into other branches. Check out some [examples in the tests](__tests__/utils.test.ts).

`jira-lint` already skips PRs which are filed by bots (for eg. [dependabot](https://github.com/marketplace/dependabot-preview)). You can add more bots to [this list](src/constants.ts), or add the branch-format followed by the bot PRs to the `skip-branches` option.

## Contributing

### Prerequisites

- Node.js 22 (see `.nvmrc`). Use `nvm use` to switch.

### Setup

```bash
npm ci
```

### Development workflow

```bash
npm run lint      # prettier + eslint
npm run test      # jest
npm run build     # bundle src/main.ts -> lib/index.js via ncc
```

### Important: build before you push

GitHub Actions runs the **compiled** `lib/index.js` bundle directly — not the TypeScript source. After making changes to any file in `src/`, you **must** run:

```bash
npm run build
git add lib
```

and commit the updated `lib/` before pushing. CI will fail if the committed `lib/` doesn't match what `npm run build` produces.

## FAQ

<details>
  <summary>Why is a Jira key required in the branch names?</summary>

The key is required in order to:

- Automate change-logs and release notes.
- Automate alerts to QA/Product teams and other external stake-holders.
- Help us retrospect the sprint progress.

</details>

<details>
  <summary>Is there a way to get around this?</summary>
  Nope.
</details>

[jira]: https://www.atlassian.com/software/jira
