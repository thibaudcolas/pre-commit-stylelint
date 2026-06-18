---
name: usage-report
description: Generate and publish the yearly mirror maintenance and usage report for pre-commit-stylelint, automating data collection from GitHub APIs, git history, and workflow runs.
license: MIT
---

# Usage Report: pre-commit-stylelint

Automatically generate the yearly maintenance report for this pre-commit mirror repo and publish it as a comment on [Discussion #1](https://github.com/thibaudcolas/pre-commit-stylelint/discussions/1).

## Prerequisites

- `gh` CLI authenticated with a token that has `repo` and `discussions` scopes
- `GITHUB_TOKEN` environment variable set (or `gh` already authenticated)
- This repo checked out locally on `main`

## Data sources

| Data point                    | Source                        | Command                                                                     |
| ----------------------------- | ----------------------------- | --------------------------------------------------------------------------- |
| Last report date | Fetch last comment on Discussion #1 | `gh api repos/:owner/:repo/discussions/1/comments` (last item) |
| Commits since last report     | `git log`                     | `git log --oneline --since="<last_report_date>"`                            |
| Automated commits (exclude)   | Filter by author              | `git log --author="Github Actions"`                                         |
| Tags published                | `git tag`                     | `git tag --sort=creatordate`                                                |
| Workflow run count & duration | GH Actions API                | `gh api repos/:owner/:repo/actions/runs?event=schedule&created=>2025-07-20` |
| Run timing                    | GH Actions Timing API         | `gh api repos/:owner/:repo/actions/runs/:id/timing`                         |
| Traffic clones (14d)          | GH Traffic API                | `gh api repos/:owner/:repo/traffic/clones`                                  |
| Traffic views (14d)           | GH Traffic API                | `gh api repos/:owner/:repo/traffic/views`                                   |
| Post to discussion            | GH Discussions API            | `gh api repos/:owner/:repo/discussions/1/comments`                          |

## Methodology

### 1. Determine last report date

Fetch the last comment from Discussion #1 via `gh api repos/thibaudcolas/pre-commit-stylelint/discussions/1/comments`. Use the comment's `created_at` date as the "since" date. Default to one year ago if no comments exist.

### 2. Maintenance time

- Count all commits since the last report date: `git log --oneline --since="<last_report_date>"`
- Exclude automated commits (author contains "Github Actions" or message starts with "Mirror:")
- Count remaining manual commits
- Extrapolate: assume 15min per manual commit for maintenance tasks (writing report, reviewing issues, adjusting config). Cap at "about 1h" since actual effort is minimal.

### 3. Infrastructure: workflow execution time

- Fetch all scheduled workflow runs since the last report date:
  ```
  gh api "repos/thibaudcolas/pre-commit-stylelint/actions/runs?event=schedule&per_page=100"
  ```
- Page through results if `total_count > 100`.
- For each successful run, fetch timing:
  ```
  gh api "repos/thibaudcolas/pre-commit-stylelint/actions/runs/<id>/timing"
  ```
  Extract `run_duration_ms`.
- Sum all durations, calculate average per run, total run time in minutes.
- Count total successful runs since last report.
- Note the schedule frequency from `.github/workflows/mirror.yml` (currently `38 6 * * 1,4` = Mon & Thu = 2x/week).

### 4. Infrastructure: tags published

- List all tags created since the last report date:
  ```
  git tag --sort=creatordate
  ```
- For each tag, check its creation date with `git log --format="%ci" -1 <tag>`.
- Count tags in the date range.
- Identify oldest and newest tag in the range (by version number).
- Also count total tags in the repo: `git tag | wc -l`.

### 5. Usage: traffic analytics

- Fetch 14-day clone data:
  ```
  gh api repos/thibaudcolas/pre-commit-stylelint/traffic/clones
  ```
- Fetch 14-day view data:
  ```
  gh api repos/thibaudcolas/pre-commit-stylelint/traffic/views
  ```
- Extract `count` and `uniques` from the `clones` array (14 data points).
- Calculate daily averages: `sum(counts) / 14`, `sum(uniques) / 14`.
- Extrapolate to yearly: `daily_avg * 365`.
- Round to nearest hundred for presentation (e.g. "250k installs per year").

### 6. Compose the report

Format the report in markdown following the style of `maintenance-reports.md`:

```markdown
# Usage Report: <current year>

## Maintenance

- <extrapolated time> of maintenance since last report.
- <manual commits> manual commits (excluding automated mirror commits).

## Infrastructure

- <runs> scheduled workflow runs since <last report date>, averaging <avg_duration>s each. Total ~<total_minutes>min of run time.
- <tags_count> versions published since the last report, from <oldest_tag> to <newest_tag> (<total_tags> total tags in repo).

## Usage

- <daily_avg_uniques> people or CI builds using the mirror per day (unique clones).
- <daily_avg_clones> clones per day on average.
- <daily_avg_views> page views per day.
- Extrapolated: ~<yearly_uniques> unique installations per year, ~<yearly_clones> clones per year.

## Takeaway

<brief reflection>
```

Append the raw traffic data in a details/summary block for reference.

Include the current date and data collection timestamp.

### 7. Publish

Post the report as a comment on Discussion #1:

```
gh api repos/thibaudcolas/pre-commit-stylelint/discussions/1/comments \
  -f body='<report markdown>'
```

## Caveats

- Traffic API only exposes 14 days of data; yearly extrapolation assumes constant traffic throughout the year.
- Workflow run durations are from `run_duration_ms` which includes queue time, not just execution time. The timing API shows total wall-clock time.
- The "Mirror:" commits are all from Github Actions bot; the `pre-commit-mirror-maker` tool auto-commits when it detects a new version on npm.
- Manual commits are rare; mostly this README and maintenance-reports.md updates.

## Example output

For reference, see the existing reports in Discussion #1.
