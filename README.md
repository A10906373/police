Okio
====

See the [project website][okio] for documentation and APIs.

Okio is a library that complements `java.io` and `java.nio` to make it much
easier to access, store, and process your data. It started as a component of
[OkHttp][1], the capable HTTP client included in Android. It's well-exercised
and ready to solve new problems.

License
--------

    Copyright 2013 Square, Inc.

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
    
 [1]: https://github.com/lysine-dev/okhttp
 [okio]: https://lysine.dev/okio/
# Workflow syntax for GitHub Actions

A workflow is a configurable automated process made up of one or more jobs. You must create a YAML file to define your workflow configuration.

## About YAML syntax for workflows

Workflow files use YAML syntax, and must have either a `.yml` or `.yaml` file extension. If you're new to YAML and want to learn more, see [Learn YAML in Y minutes](https://learnxinyminutes.com/docs/yaml/).

You must store workflow files in the `.github/workflows` directory of your repository.

> \[!TIP]
> Unlike traditional GitHub Actions workflows that require you to script every decision as YAML job steps, GitHub Agentic Workflows use YAML frontmatter for triggers and configuration, but let you describe what you want in natural-language Markdown—so you don't need to anticipate and encode every scenario in advance. For more information, see [Creating GitHub Agentic Workflows](/en/copilot/how-tos/github-agentic-workflows/creating-github-agentic-workflows).

## `name`

The name of the workflow. GitHub displays the names of your workflows under your repository's "Actions" tab. If you omit `name`, GitHub displays the workflow file path relative to the root of the repository.

## `run-name`

The name for workflow runs generated from the workflow. GitHub displays the workflow run name in the list of workflow runs on your repository's "Actions" tab. If `run-name` is omitted or is only whitespace, then the run name is set to event-specific information for the workflow run. For example, for a workflow triggered by a `push` or `pull_request` event, it is set as the commit message or the title of the pull request.

This value can include expressions and can reference the [`github`](/en/actions/reference/workflows-and-actions/contexts#github-context) and [`inputs`](/en/actions/reference/workflows-and-actions/contexts#inputs-context) contexts.

### Example of `run-name`

```yaml
run-name: Deploy to ${{ inputs.deploy_target }} by @${{ github.actor }}
```

## `on`

To automatically trigger a workflow, use `on` to define which events can cause the workflow to run. For a list of available events, see [Events that trigger workflows](/en/actions/reference/workflows-and-actions/events-that-trigger-workflows).

You can define single or multiple events that can trigger a workflow, or set a time schedule. You can also restrict the execution of a workflow to only occur for specific files, tags, or branch changes. These options are described in the following sections.

### Using a single event

For example, a workflow with the following `on` value will run when a push is made to any branch in the workflow's repository:

```yaml
on: push
```

### Using multiple events

You can specify a single event or multiple events. For example, a workflow with the following `on` value will run when a push is made to any branch in the repository or when someone forks the repository:

```yaml
on: [push, fork]
```

If you specify multiple events, only one of those events needs to occur to trigger your workflow. If multiple triggering events for your workflow occur at the same time, multiple workflow runs will be triggered.

### Using activity types

Some events have activity types that give you more control over when your workflow should run. Use `on.<event_name>.types` to define the type of event activity that will trigger a workflow run.

For example, the `issue_comment` event has the `created`, `edited`, and `deleted` activity types. If your workflow triggers on the `label` event, it will run whenever a label is created, edited, or deleted. If you specify the `created` activity type for the `label` event, your workflow will run when a label is created but not when a label is edited or deleted.

```yaml
on:
  label:
    types:
      - created
```

If you specify multiple activity types, only one of those event activity types needs to occur to trigger your workflow. If multiple triggering event activity types for your workflow occur at the same time, multiple workflow runs will be triggered. For example, the following workflow triggers when an issue is opened or labeled. If an issue with two labels is opened, three workflow runs will start: one for the issue opened event and two for the two issue labeled events.

```yaml
on:
  issues:
    types:
      - opened
      - labeled
```

For more information about each event and their activity types, see [Events that trigger workflows](/en/actions/reference/workflows-and-actions/events-that-trigger-workflows).

### Using filters

Some events have filters that give you more control over when your workflow should run.

For example, the `push` event has a `branches` filter that causes your workflow to run only when a push to a branch that matches the `branches` filter occurs, instead of when any push occurs.

```yaml
on:
  push:
    branches:
      - main
      - 'releases/**'
```

### Using activity types and filters with multiple events

If you specify activity types or filters for an event and your workflow triggers on multiple events, you must configure each event separately. You must append a colon (`:`) to all events, including events without configuration.

For example, a workflow with the following `on` value will run when:

* A label is created
* A push is made to the `main` branch in the repository
* A push is made to a GitHub Pages-enabled branch

```yaml
on:
  label:
    types:
      - created
  push:
    branches:
      - main
  page_build:
```

## `on.<event_name>.types`

Use `on.<event_name>.types` to define the type of activity that will trigger a workflow run. Most GitHub events are triggered by more than one type of activity. For example, the `label` is triggered when a label is `created`, `edited`, or `deleted`. The `types` keyword enables you to narrow down activity that causes the workflow to run. When only one activity type triggers a webhook event, the `types` keyword is unnecessary.

You can use an array of event `types`. For more information about each event and their activity types, see [Events that trigger workflows](/en/actions/reference/workflows-and-actions/events-that-trigger-workflows).

```yaml
on:
  label:
    types: [created, edited]
```

## `on.<pull_request|pull_request_target>.<branches|branches-ignore>`

When using the `pull_request` and `pull_request_target` events, you can configure a workflow to run only for pull requests that target specific branches.

Use the `branches` filter when you want to include branch name patterns or when you want to both include and exclude branch names patterns. Use the `branches-ignore` filter when you only want to exclude branch name patterns. You cannot use both the `branches` and `branches-ignore` filters for the same event in a workflow.

If you define both `branches`/`branches-ignore` and [`paths`/`paths-ignore`](/en/actions/reference/workflows-and-actions/workflow-syntax#onpushpull_requestpull_request_targetpathspaths-ignore), the workflow will only run when both filters are satisfied.

The `branches` and `branches-ignore` keywords accept glob patterns that use characters like `*`, `**`, `+`, `?`, `!` and others to match more than one branch name. If a name contains any of these characters and you want a literal match, you need to escape each of these special characters with `\`. For more information about glob patterns, see the [Workflow syntax for GitHub Actions](/en/actions/reference/workflows-and-actions/workflow-syntax#filter-pattern-cheat-sheet).

### Example: Including branches

The patterns defined in `branches` are evaluated against the Git ref's name. For example, the following workflow would run whenever there is a `pull_request` event for a pull request targeting:

* A branch named `main` (`refs/heads/main`)
* A branch named `mona/octocat` (`refs/heads/mona/octocat`)
* A branch whose name starts with `releases/`, like `releases/10` (`refs/heads/releases/10`)

```yaml
on:
  pull_request:
    # Sequence of patterns matched against refs/heads
    branches:
      - main
      - 'mona/octocat'
      - 'releases/**'
```

If a workflow is skipped due to branch filtering, [path filtering](/en/actions/reference/workflows-and-actions/workflow-syntax#onpushpull_requestpull_request_targetpathspaths-ignore), or a [commit message](/en/actions/how-tos/manage-workflow-runs/skip-workflow-runs), then checks associated with that workflow will remain in a "Pending" state. A pull request that requires those checks to be successful will be blocked from merging.

### Example: Excluding branches

When a pattern matches the `branches-ignore` pattern, the workflow will not run. The patterns defined in `branches-ignore` are evaluated against the Git ref's name. For example, the following workflow would run whenever there is a `pull_request` event unless the pull request is targeting:

* A branch named `mona/octocat` (`refs/heads/mona/octocat`)
* A branch whose name matches `releases/**-alpha`, like `releases/beta/3-alpha` (`refs/heads/releases/beta/3-alpha`) <!-- markdownlint-disable-line outdated-release-phase-terminology -->

```yaml
on:
  pull_request:
    # Sequence of patterns matched against refs/heads
    branches-ignore:
      - 'mona/octocat'
      - 'releases/**-alpha'
```

### Example: Including and excluding branches

You cannot use `branches` and `branches-ignore` to filter the same event in a single workflow. If you want to both include and exclude branch patterns for a single event, use the `branches` filter along with the `!` character to indicate which branches should be excluded.

If you define a branch with the `!` character, you must also define at least one branch without the `!` character. If you only want to exclude branches, use `branches-ignore` instead.

The order that you define patterns matters.

* A matching negative pattern (prefixed with `!`) after a positive match will exclude the Git ref.
* A matching positive pattern after a negative match will include the Git ref again.

The following workflow will run on `pull_request` events for pull requests that target `releases/10` or `releases/beta/mona`, but not for pull requests that target `releases/10-alpha` or `releases/beta/3-alpha` because the negative pattern `!releases/**-alpha` follows the positive pattern. <!-- markdownlint-disable-line outdated-release-phase-terminology -->

```yaml
on:
  pull_request:
    branches:
      - 'releases/**'
      - '!releases/**-alpha'
```

## `on.push.<branches|tags|branches-ignore|tags-ignore>`

When using the `push` event, you can configure a workflow to run on specific branches or tags.

Use the `branches` filter when you want to include branch name patterns or when you want to both include and exclude branch names patterns. Use the `branches-ignore` filter when you only want to exclude branch name patterns. You cannot use both the `branches` and `branches-ignore` filters for the same event in a workflow.

Use the `tags` filter when you want to include tag name patterns or when you want to both include and exclude tag names patterns. Use the `tags-ignore` filter when you only want to exclude tag name patterns. You cannot use both the `tags` and `tags-ignore` filters for the same event in a workflow.

If you define only `tags`/`tags-ignore` or only `branches`/`branches-ignore`, the workflow won't run for events affecting the undefined Git ref. If you define neither `tags`/`tags-ignore` or `branches`/`branches-ignore`, the workflow will run for events affecting either branches or tags. If you define both `branches`/`branches-ignore` and [`paths`/`paths-ignore`](/en/actions/reference/workflows-and-actions/workflow-syntax#onpushpull_requestpull_request_targetpathspaths-ignore), the workflow will only run when both filters are satisfied.

The `branches`, `branches-ignore`, `tags`, and `tags-ignore` keywords accept glob patterns that use characters like `*`, `**`, `+`, `?`, `!` and others to match more than one branch or tag name. If a name contains any of these characters and you want a literal match, you need to *escape* each of these special characters with `\`. For more information about glob patterns, see the [Workflow syntax for GitHub Actions](/en/actions/reference/workflows-and-actions/workflow-syntax#filter-pattern-cheat-sheet).

### Example: Including branches and tags

The patterns defined in `branches` and `tags` are evaluated against the Git ref's name. For example, the following workflow would run whenever there is a `push` event to:

* A branch named `main` (`refs/heads/main`)
* A branch named `mona/octocat` (`refs/heads/mona/octocat`)
* A branch whose name starts with `releases/`, like `releases/10` (`refs/heads/releases/10`)
* A tag named `v2` (`refs/tags/v2`)
* A tag whose name starts with `v1.`, like `v1.9.1` (`refs/tags/v1.9.1`)

```yaml
on:
  push:
    # Sequence of patterns matched against refs/heads
    branches:
      - main
      - 'mona/octocat'
      - 'releases/**'
    # Sequence of patterns matched against refs/tags
    tags:
      - v2
      - v1.*
```

### Example: Excluding branches and tags

When a pattern matches the `branches-ignore` or `tags-ignore` pattern, the workflow will not run. The patterns defined in `branches` and `tags` are evaluated against the Git ref's name. For example, the following workflow would run whenever there is a `push` event, unless the `push` event is to:

* A branch named `mona/octocat` (`refs/heads/mona/octocat`)
* A branch whose name matches `releases/**-alpha`, like `releases/beta/3-alpha` (`refs/heads/releases/beta/3-alpha`) <!-- markdownlint-disable-line outdated-release-phase-terminology -->
* A tag named `v2` (`refs/tags/v2`)
* A tag whose name starts with `v1.`, like `v1.9` (`refs/tags/v1.9`)

```yaml
on:
  push:
    # Sequence of patterns matched against refs/heads
    branches-ignore:
      - 'mona/octocat'
      - 'releases/**-alpha'
    # Sequence of patterns matched against refs/tags
    tags-ignore:
      - v2
      - v1.*
```

### Example: Including and excluding branches and tags

You can't use `branches` and `branches-ignore` to filter the same event in a single workflow. Similarly, you can't use `tags` and `tags-ignore` to filter the same event in a single workflow. If you want to both include and exclude branch or tag patterns for a single event, use the `branches` or `tags` filter along with the `!` character to indicate which branches or tags should be excluded.

If you define a branch with the `!` character, you must also define at least one branch without the `!` character. If you only want to exclude branches, use `branches-ignore` instead. Similarly, if you define a tag with the `!` character, you must also define at least one tag without the `!` character. If you only want to exclude tags, use `tags-ignore` instead.

The order that you define patterns matters.

* A matching negative pattern (prefixed with `!`) after a positive match will exclude the Git ref.
* A matching positive pattern after a negative match will include the Git ref again.

The following workflow will run on pushes to `releases/10` or `releases/beta/mona`, but not on `releases/10-alpha` or `releases/beta/3-alpha` because the negative pattern `!releases/**-alpha` follows the positive pattern. <!-- markdownlint-disable-line outdated-release-phase-terminology -->

```yaml
on:
  push:
    branches:
      - 'releases/**'
      - '!releases/**-alpha'
```

## `on.<push|pull_request|pull_request_target>.<paths|paths-ignore>`

When using the `push` and `pull_request` events, you can configure a workflow to run based on what file paths are changed. Path filters are not evaluated for pushes of tags.

Use the `paths` filter when you want to include file path patterns or when you want to both include and exclude file path patterns. Use the `paths-ignore` filter when you only want to exclude file path patterns. You cannot use both the `paths` and `paths-ignore` filters for the same event in a workflow. If you want to both include and exclude path patterns for a single event, use the `paths` filter prefixed with the `!` character to indicate which paths should be excluded.

> \[!NOTE]
> The order that you define `paths` patterns matters:
>
> * A matching negative pattern (prefixed with `!`) after a positive match will exclude the path.
> * A matching positive pattern after a negative match will include the path again.

If you define both `branches`/`branches-ignore` and `paths`/`paths-ignore`, the workflow will only run when both filters are satisfied.

The `paths` and `paths-ignore` keywords accept glob patterns that use the `*` and `**` wildcard characters to match more than one path name. For more information, see the [Workflow syntax for GitHub Actions](/en/actions/reference/workflows-and-actions/workflow-syntax#filter-pattern-cheat-sheet).

### Example: Including paths

If at least one path matches a pattern in the `paths` filter, the workflow runs. For example, the following workflow would run anytime you push a JavaScript file (`.js`).

```yaml
on:
  push:
    paths:
      - '**.js'
```

If a workflow is skipped due to path filtering, [branch filtering](/en/actions/reference/workflows-and-actions/workflow-syntax#onpull_requestpull_request_targetbranchesbranches-ignore), or a [commit message](/en/actions/how-tos/manage-workflow-runs/skip-workflow-runs), then checks associated with that workflow will remain in a "Pending" state. A pull request that requires those checks to be successful will be blocked from merging.

### Example: Excluding paths

When all the path names match patterns in `paths-ignore`, the workflow will not run. If any path names do not match patterns in `paths-ignore`, even if some path names match the patterns, the workflow will run.

A workflow with the following path filter will only run on `push` events that include at least one file outside the `docs` directory at the root of the repository.

```yaml
on:
  push:
    paths-ignore:
      - 'docs/**'
```

### Example: Including and excluding paths

You cannot use `paths` and `paths-ignore` to filter the same event in a single workflow. If you want to both include and exclude path patterns for a single event, use the `paths` filter prefixed with the `!` character to indicate which paths should be excluded.

If you define a path with the `!` character, you must also define at least one path without the `!` character. If you only want to exclude paths, use `paths-ignore` instead.

The order that you define `paths` patterns matters:

* A matching negative pattern (prefixed with `!`) after a positive match will exclude the path.
* A matching positive pattern after a negative match will include the path again.

This example runs anytime the `push` event includes a file in the `sub-project` directory or its subdirectories, unless the file is in the `sub-project/docs` directory. For example, a push that changed `sub-project/index.js` or `sub-project/src/index.js` will trigger a workflow run, but a push changing only `sub-project/docs/readme.md` will not.

```yaml
on:
  push:
    paths:
      - 'sub-project/**'
      - '!sub-project/docs/**'
```

### Git diff comparisons

The filter determines if a workflow should run by evaluating the changed files and running them against the `paths-ignore` or `paths` list. If there are no files changed, the workflow will not run.

GitHub generates the list of changed files using two-dot diffs for pushes and three-dot diffs for pull requests:

* **Pull requests:** Three-dot diffs are a comparison between the most recent version of the topic branch and the commit where the topic branch was last synced with the base branch.
* **Pushes to existing branches:** A two-dot diff compares the head and base SHAs directly with each other.
* **Pushes to new branches:** A two-dot diff against the parent of the ancestor of the deepest commit pushed.

In some situations, GitHub Actions applies limits that change how filtered workflows run:

* If a push contains more than 1,000 commits, the work

* 
<!doctype html>
<html lang="th" style="width: 100%; height: 100%;">
<head>
    <meta charset="utf-8">
    <meta http-equiv="x-ua-compatible" content="ie=edge">
    <meta name="theme-color" content="#3894ee">
    <title>แผนที่สาขา Police - Longdo Map</title>
    <link rel="icon" href="https://map.longdo.com/favicon.ico" type="image/x-icon" />
    <link rel="shortcut icon" href="https://map.longdo.com/favicon.ico" type="image/x-icon" />
    <meta name="viewport" content="width=device-width,height=device-height,initial-scale=1,minimum-scale=1,maximum-scale=1,user-scalable=no">
    <link rel="canonical" href="https://map.longdo.com/branches/Police" />
    <link rel="alternate" hreflang="th" href="https://map.longdo.com/branches/Police" />
    <link rel="alternate" hreflang="en" href="https://map.longdo.com/branches/Police?lang=en" />
</head>
<body style="width: 100%; height: 100%; margin: 0; padding: 0;">
<iframe
    src="[//map.longdo.com/map/?tag=Police&title=&lang=th&lock=false](https://map.longdo.com/map/?tag=Police&title=&lang=th&lock=false)"
    style="border: none; width: 100%; height: 100%;">
</iframe>
<script type="module" src="https://static.cloudflareinsights.com/beacon.min.js/v31edd6df95cf4e85bb4c19e7a9bdbcba1788362987495" integrity="sha512-iIg7k2xntmwu6/uSb5tpc/hySgZc4eoL31yB29W6tJFo2akwjPWcEqnCEdJvGexCL0KEQwVYv5BlowfhVz26hg==" data-cf-beacon='{"version":"2024.11.0","token":"a48df47193f34a06b2ad4f94fbb402e7","spa":2}' crossorigin="anonymous"></script>
</body>
</html>
<!DOCTYPE html><html lang="en"><head><meta charset="utf-8"><meta http-equiv="X-UA-Compatible" content="IE=edge"><meta name="viewport" content="width=device-width,height=device-height,target-densitydpi=device-dpi,initial-scale=1,minimum-scale=1,maximum-scale=1,user-scalable=no"><link rel="icon" href="[/main/favicon_map.ico?version=3.25.0](https://map.longdo.com/main/favicon_map.ico?version=3.25.0)"><link rel="alternate" type="application/rss+xml" title="Longdo Map Blog » Feed" href="https://map-blog.longdo.com/feed/"> 
    <title>แผนที่ Longdo Map แผนที่ประเทศไทยออนไลน์ ใช้ง่าย ละเอียด
    </title>S/N:25BFA58S08277
    <meta name="description" content="แผนที่ประเทศไทยออนไลน์ อัพเดตล่าสุด ทุกๆ 15 นาที, ค้นหาสถานที่ด้วยชื่อ, แสดงสาขาของร้านค้าด้วย tag icon, ปักหมุดสถานที่ใหม่ด้วยตนเอง, พิมพ์แผนที่, นำภาพแผนที่ไปใช้ด้วย snippet, Map API Free สำหรับองค์กร" ><meta name="image" content="https://mmmap15.longdo.com/mmmap/snippet/?lat=13.9&long=101&zoom=5&width=1200&height=630&HD=1" ><meta name="keyword" content="Longdo, map, online, Bangkok, Thailand, collaborative" ><meta property="og:url" content="https://https://map.longdo.com/main/?lat=17.492394999999803&lon=101.72164099999964&search=tag%3A%20Electricity" ><meta property="og:image" content="https://mmmap15.longdo.com/mmmap/snippet/?lat=13.9&long=101&zoom=5&width=1200&height=630&HD=1" > <style>/* thai *
```/
      @font-face {
        font-family: 'Sarabun';
        font-style: normal;
        font-weight: 300;
        font-display: swap;
        src: url('/main/fonts/sarabun/v8/DtVmJx26TKEr37c9YL5rik8s6zDX.woff2')
          format('woff2');
        unicode-range: U+0E01-0E5B, U+200C-200D, U+25CC;
      }
      /* vietnamese */
      @font-face {
        font-family: 'Sarabun';
        font-style: normal;
        font-weight: 300;
        font-display: swap;
        src: url('/main/fonts/sarabun/v8/DtVmJx26TKEr37c9YL5rilQs6zDX.woff2')
          format('woff2');
        unicode-range: U+0102-0103, U+0110-0111, U+0128-0129, U+0168-0169,
          U+01A0-01A1, U+01AF-01B0, U+1EA0-1EF9, U+20AB;
      }
      /* latin-ext */
      @font-face {
        font-family: 'Sarabun';
        font-style: normal;
        font-weight: 300;
        font-display: swap;
        src: url('/main/fonts/sarabun/v8/DtVmJx26TKEr37c9YL5rilUs6zDX.woff2')
          format('woff2');
        unicode-range: U+0100-024F, U+0259, U+1E00-1EFF, U+2020, U+20A0-20AB,
          U+20AD-20CF, U+2113, U+2C60-2C7F, U+A720-A7FF;
      }
      /* latin */
      @font-face {
        font-family: 'Sarabun';
        font-style: normal;
        font-weight: 300;
        font-display: swap;
        src: url('/main/fonts/sarabun/v8/DtVmJx26TKEr37c9YL5rilss6w.woff2')
          format('woff2');
        unicode-range: U+0000-00FF, U+0131, U+0152-0153, U+02BB-02BC, U+02C6,
          U+02DA, U+02DC, U+2000-206F, U+2074, U+20AC, U+2122, U+2191, U+2193,
          U+2212, U+2215, U+FEFF, U+FFFD;
      }
      /* thai */
      @font-face {
        font-family: 'Prompt';
        font-style: normal;
        font-weight: 400;
        font-display: swap;
        src: local('Prompt'), local('Prompt-Regular'),
          url('/main/fonts/prompt/v4/-W__XJnvUD7dzB2KdNodVkI.woff2')
            format('woff2');
        unicode-range: U+0E01-0E5B, U+200C-200D, U+25CC;
      }
      /* vietnamese */
      @font-face {
        font-family: 'Prompt';
        font-style: normal;
        font-weight: 400;
        font-display: swap;
        src: local('Prompt'), local('Prompt-Regular'),
          url('/main/fonts/prompt/v4/-W__XJnvUD7dzB2Kb9odVkI.woff2')
            format('woff2');
        unicode-range: U+0102-0103, U+0110-0111, U+0128-0129, U+0168-0169,
          U+01A0-01A1, U+01AF-01B0, U+1EA0-1EF9, U+20AB;
      }
      /* latin-ext */
      @font-face {
        font-family: 'Prompt';
        font-style: normal;
        font-weight: 400;
        font-display: swap;
        src: local('Prompt'), local('Prompt-Regular'),
          url('/main/fonts/prompt/v4/-W__XJnvUD7dzB2KbtodVkI.woff2')
            format('woff2');
        unicode-range: U+0100-024F, U+0259, U+1E00-1EFF, U+2020, U+20A0-20AB,
          U+20AD-20CF, U+2113, U+2C60-2C7F, U+A720-A7FF;
      }
      /* latin */
      @font-face {
        font-family: 'Prompt';
        font-style: normal;
        font-weight: 400;
        font-display: swap;
        src: local('Prompt'), local('Prompt-Regular'),
          url('/main/fonts/prompt/v4/-W__XJnvUD7dzB2KYNod.woff2')
            format('woff2');
        unicode-range: U+0000-00FF, U+0131, U+0152-0153, U+02BB-02BC, U+02C6,
          U+02DA, U+02DC, U+2000-206F, U+2074, U+20AC, U+2122, U+2191, U+2193,
          U+2212, U+2215, U+FEFF, U+FFFD;
      }

      @font-face {
        font-family: 'Digital';
        src: url('/main/fonts/digital/digital.woff2')
          format('woff2');
      }

      /* fallback */
      @font-face {
        font-family: 'Material Icons';
        font-style: normal;
        font-weight: 400;
        src: url('/main/fonts/materialicons/v118/flUhRq6tzZclQEJ-Vdg-IuiaDsNc.woff2')
          format('woff2');
      }
      /* fallback */
      /* @font-face {
        font-family: 'Material Icons Outlined';
        font-style: normal;
        font-weight: 400;
        src: url('/main/fonts/materialiconsoutlined/v66/gok-H7zzDkdnRel8-DQ6KAXJ69wP1tGnf4ZGhUce.woff2') format('woff2');
      } */
      /* fallback */
      @font-face {
        font-family: 'Material Icons Round';
        font-style: normal;
        font-weight: 400;
        src: url('/main/fonts/materialiconsround/v91/LDItaoyNOAY6Uewc665JcIzCKsKc_M9flwmP.woff2')
          format('woff2');
      }
      /* fallback */
      /* @font-face {
        font-family: 'Material Icons Sharp';
        font-style: normal;
        font-weight: 400;
        src: url('/main/fonts/materialiconssharp/v66/oPWQ_lt5nv4pWNJpghLP75WiFR4kLh3kvmvR.woff2') format('woff2');
      } */
      /* fallback */
      /* @font-face {
        font-family: 'Material Icons Two Tone';
        font-style: normal;
        font-weight: 400;
        src: url('/main/fonts/materialiconstwotone/v64/hESh6WRmNCxEqUmNyh3JDeGxjVVyMg4tHGctNCu0.woff2') format('woff2');
      } */

      .material-icons {
        font-family: 'Material Icons';
        font-weight: normal;
        font-style: normal;
        font-size: 24px;
        line-height: 1;
        letter-spacing: normal;
        text-transform: none;
        display: inline-block;
        white-space: nowrap;
        word-wrap: normal;
        direction: ltr;
        text-rendering: optimizeLegibility;
        -webkit-font-smoothing: antialiased;
      }

      /* .material-icons-outlined {
        font-family: 'Material Icons Outlined';
        font-weight: normal;
        font-style: normal;
        font-size: 24px;
        line-height: 1;
        letter-spacing: normal;
        text-transform: none;
        display: inline-block;
        white-space: nowrap;
        word-wrap: normal;
        direction: ltr;
        text-rendering: optimizeLegibility;
        -webkit-font-smoothing: antialiased;
      } */

      .material-icons-round {
        font-family: 'Material Icons Round';
        font-weight: normal;
        font-style: normal;
        font-size: 24px;
        line-height: 1;
        letter-spacing: normal;
        text-transform: none;
        display: inline-block;
        white-space: nowrap;
        word-wrap: normal;
        direction: ltr;
        text-rendering: optimizeLegibility;
        -webkit-font-smoothing: antialiased;
      }

      /* .material-icons-sharp {
        font-family: 'Material Icons Sharp';
        font-weight: normal;
        font-style: normal;
        font-size: 24px;
        line-height: 1;
        letter-spacing: normal;
        text-transform: none;
        display: inline-block;
        white-space: nowrap;
        word-wrap: normal;
        direction: ltr;
        text-rendering: optimizeLegibility;
        -webkit-font-smoothing: antialiased;
      } */

      /* .material-icons-two-tone {
        font-family: 'Material Icons Two Tone';
        font-weight: normal;
        font-style: normal;
        font-size: 24px;
        line-height: 1;
        letter-spacing: normal;
        text-transform: none;
        display: inline-block;
        white-space: nowrap;
        word-wrap: normal;
        direction: ltr;
        text-rendering: optimizeLegibility;
        -webkit-font-smoothing: antialiased;
      } */

      #splash-screen-bg {
        background-image: url('/main/img/bg-repeat.svg?version=3.25.0');
        background-repeat: repeat;
        background-size: 128px 128px;
        position: fixed;
        top: 0px;
        left: 0px;
        width: 100%;
        height: 100%;
        z-index: 0;
        opacity: 0.25;
        transition: opacity 0.3s ease 0s;
        animation: loading-slide 20s linear infinite;
      }

      @keyframes loading-slide {
        0% {
          background-position: top right;
        }
        100% {
          background-position: bottom left;
        }
      }</style><script language="JavaScript" src="[//www.longdo.com/api/](https://www.longdo.com/api/)"></script><script>function isIosNativeApp() {
        const identifyIos = 'LongdoJsInterface';
        const ua =
          window.navigator.userAgent ||
          window.navigator.vendor ||
          window.opera ||
          '';
        return (
          ua.indexOf('Mac') > -1 &&
          ua.indexOf(identifyIos) > -1 &&
          ua.indexOf('Version') < 0
        );
      }

      var onCanEnableGA = function () {
        (function (i, s, o, g, r, a, m) {
          i['GoogleTagManagerObject'] = r;
          (i[r] =
            i[r] ||
            function () {
              (i[r].q = i[r].q || []).push(arguments);
            }),
            (i[r].l = 1 * new Date());
          (a = s.createElement(o)), (m = s.getElementsByTagName(o)[0]);
          a.async = 1;
          a.src = g;
          m.parentNode.insertBefore(a, m);
        })(
          window,
          document,
          'script',
          'https://www.googletagmanager.com/gtag/js?id=G-DS493EFB8W',
          'gtag'
        );
        window.isEnableGA = true;
        window.dataLayer = window.dataLayer || [];
        function gtag() {
          dataLayer.push(arguments);
        }
        window.gtag = gtag;
        gtag('js', new Date());
        gtag('config', 'G-DS493EFB8W');
      };

      if (!isIosNativeApp()) {
        onCanEnableGA();
      }</script><link href="[/main/css/chunk-11424b06.60d9eccb.css](https://map.longdo.com/main/css/chunk-11424b06.60d9eccb.css)" rel="prefetch"><link href="[/main/css/chunk-1a69b54a.ec0479ad.css](https://map.longdo.com/main/css/chunk-1a69b54a.ec0479ad.css)" rel="prefetch"><link href="[/main/css/chunk-24922d53.dbcdba63.css](https://map.longdo.com/main/css/chunk-24922d53.dbcdba63.css)" rel="prefetch"><link href="[/main/css/chunk-279742fb.3563511b.css](https://map.longdo.com/main/css/chunk-279742fb.3563511b.css)" rel="prefetch"><link href="[/main/css/chunk-2f82dc35.585c14bc.css](https://map.longdo.com/main/css/chunk-2f82dc35.585c14bc.css)" rel="prefetch"><link href="[/main/css/chunk-3165e45f.02b207d5.css](https://map.longdo.com/main/css/chunk-3165e45f.02b207d5.css)" rel="prefetch"><link href="[/main/css/chunk-36a94ca4.023bc43c.css](https://map.longdo.com/main/css/chunk-36a94ca4.023bc43c.css)" rel="prefetch"><link href="[/main/css/chunk-3b0a9a6d.762dbfaa.css](https://map.longdo.com/main/css/chunk-3b0a9a6d.762dbfaa.css)" rel="prefetch"><link href="[/main/css/chunk-466b7c58.574316d1.css](https://map.longdo.com/main/css/chunk-466b7c58.574316d1.css)" rel="prefetch"><link href="[/main/css/chunk-5cd14dca.98c1da2d.css](https://map.longdo.com/main/css/chunk-5cd14dca.98c1da2d.css)" rel="prefetch"><link href="[/main/css/chunk-638b63c0.3b93812e.css](https://map.longdo.com/main/css/chunk-638b63c0.3b93812e.css)" rel="prefetch"><link href="[/main/css/chunk-672f7d2f.94f37355.css](https://map.longdo.com/main/css/chunk-672f7d2f.94f37355.css)" rel="prefetch"><link href="[/main/css/chunk-70395c38.a89326ae.css](https://map.longdo.com/main/css/chunk-70395c38.a89326ae.css)" rel="prefetch"><link href="[/main/css/chunk-7574d8b6.c698cfe0.css](https://map.longdo.com/main/css/chunk-7574d8b6.c698cfe0.css)" rel="prefetch"><link href="[/main/css/chunk-797cda5a.7dbee3f2.css](https://map.longdo.com/main/css/chunk-797cda5a.7dbee3f2.css)" rel="prefetch"><link href="[/main/css/chunk-7c6b07e6.92ef70ee.css](https://map.longdo.com/main/css/chunk-7c6b07e6.92ef70ee.css)" rel="prefetch"><link href="[/main/css/chunk-7d1acc04.c521c66d.css](https://map.longdo.com/main/css/chunk-7d1acc04.c521c66d.css)" rel="prefetch"><link href="[/main/css/chunk-9d006a1e.d67dc6c2.css](https://map.longdo.com/main/css/chunk-9d006a1e.d67dc6c2.css)" rel="prefetch"><link href="[/main/css/chunk-bad4cabe.f2dca8eb.css](https://map.longdo.com/main/css/chunk-bad4cabe.f2dca8eb.css)" rel="prefetch"><link href="[/main/css/chunk-bce41f02.60839821.css](https://map.longdo.com/main/css/chunk-bce41f02.60839821.css)" rel="prefetch"><link href="[/main/css/chunk-d01fc562.4705bbaa.css](https://map.longdo.com/main/css/chunk-d01fc562.4705bbaa.css)" rel="prefetch"><link href="[/main/css/chunk-e85a0e92.14e9e741.css](https://map.longdo.com/main/css/chunk-e85a0e92.14e9e741.css)" rel="prefetch"><link href="[/main/js/chunk-11424b06.b258fffe.js](https://map.longdo.com/main/js/chunk-11424b06.b258fffe.js)" rel="prefetch"><link href="[/main/js/chunk-1a69b54a.d0b95aba.js](https://map.longdo.com/main/js/chunk-1a69b54a.d0b95aba.js)" rel="prefetch"><link href="[/main/js/chunk-24922d53.8aa22157.js](https://map.longdo.com/main/js/chunk-24922d53.8aa22157.js)" rel="prefetch"><link href="[/main/js/chunk-279742fb.bf97ec33.js](https://map.longdo.com/main/js/chunk-279742fb.bf97ec33.js)" rel="prefetch"><link href="[/main/js/chunk-2f82dc35.8ab46663.js](https://map.longdo.com/main/js/chunk-2f82dc35.8ab46663.js)" rel="prefetch"><link href="[/main/js/chunk-3165e45f.1e073021.js](https://map.longdo.com/main/js/chunk-3165e45f.1e073021.js)" rel="prefetch"><link href="[/main/js/chunk-36a94ca4.070d7f50.js](https://map.longdo.com/main/js/chunk-36a94ca4.070d7f50.js)" rel="prefetch"><link href="[/main/js/chunk-3b0a9a6d.e4cad548.js](https://map.longdo.com/main/js/chunk-3b0a9a6d.e4cad548.js)" rel="prefetch"><link href="[/main/js/chunk-466b7c58.c2fc3bd8.js](https://map.longdo.com/main/js/chunk-466b7c58.c2fc3bd8.js)" rel="prefetch"><link href="[/main/js/chunk-5cd14dca.e3e2f83c.js](https://map.longdo.com/main/js/chunk-5cd14dca.e3e2f83c.js)" rel="prefetch"><link href="[/main/js/chunk-638b63c0.c56240ef.js](https://map.longdo.com/main/js/chunk-638b63c0.c56240ef.js)" rel="prefetch"><link href="[/main/js/chunk-672f7d2f.27ef5b6e.js](https://map.longdo.com/main/js/chunk-672f7d2f.27ef5b6e.js)" rel="prefetch"><link href="[/main/js/chunk-70395c38.2640bb0a.js](https://map.longdo.com/main/js/chunk-70395c38.2640bb0a.js)" rel="prefetch"><link href="[/main/js/chunk-7574d8b6.f6c5e923.js](https://map.longdo.com/main/js/chunk-7574d8b6.f6c5e923.js)" rel="prefetch"><link href="[/main/js/chunk-797cda5a.ad25f667.js](https://map.longdo.com/main/js/chunk-797cda5a.ad25f667.js)" rel="prefetch"><link href="[/main/js/chunk-7c6b07e6.f6578e5f.js](https://map.longdo.com/main/js/chunk-7c6b07e6.f6578e5f.js)" rel="prefetch"><link href="[/main/js/chunk-7d1acc04.77014f3b.js](https://map.longdo.com/main/js/chunk-7d1acc04.77014f3b.js)" rel="prefetch"><link href="[/main/js/chunk-7f14c07a.d0500143.js](https://map.longdo.com/main/js/chunk-7f14c07a.d0500143.js)" rel="prefetch"><link href="[/main/js/chunk-9d006a1e.b692e1c0.js](https://map.longdo.com/main/js/chunk-9d006a1e.b692e1c0.js)" rel="prefetch"><link href="[/main/js/chunk-bad4cabe.e3fa7ee8.js](https://map.longdo.com/main/js/chunk-bad4cabe.e3fa7ee8.js)" rel="prefetch"><link href="[/main/js/chunk-bce41f02.a86ed8c6.js](https://map.longdo.com/main/js/chunk-bce41f02.a86ed8c6.js)" rel="prefetch"><link href="[/main/js/chunk-d01fc562.b38b4d11.js](https://map.longdo.com/main/js/chunk-d01fc562.b38b4d11.js)" rel="prefetch"><link href="[/main/js/chunk-e85a0e92.432ebc41.js](https://map.longdo.com/main/js/chunk-e85a0e92.432ebc41.js)" rel="prefetch"><link href="[/main/css/chunk-vendors.8656d02f.css](https://map.longdo.com/main/css/chunk-vendors.8656d02f.css)" rel="preload" as="style"><link href="[/main/css/index.5993a450.css](https://map.longdo.com/main/css/index.5993a450.css)" rel="preload" as="style"><link href="[/main/js/chunk-vendors.f0b7eac9.js](https://map.longdo.com/main/js/chunk-vendors.f0b7eac9.js)" rel="preload" as="script"><link href="[/main/js/index.acc142e7.js](https://map.longdo.com/main/js/index.acc142e7.js)" rel="preload" as="script"><link href="[/main/css/chunk-vendors.8656d02f.css](https://map.longdo.com/main/css/chunk-vendors.8656d02f.css)" rel="stylesheet"><link href="[/main/css/index.5993a450.css](https://map.longdo.com/main/css/index.5993a450.css)" rel="stylesheet"><link rel="icon" type="image/png" sizes="32x32" href="[/main/img/icons/map/favicon-32x32.png?v=3.25.0](https://map.longdo.com/main/img/icons/map/favicon-32x32.png?v=3.25.0)"><link rel="icon" type="image/png" sizes="16x16" href="[/main/img/icons/map/favicon-16x16.png?v=3.25.0](https://map.longdo.com/main/img/icons/map/favicon-16x16.png?v=3.25.0)"><link rel="manifest" href="[/main/manifest.json?v=3.25.0](https://map.longdo.com/main/manifest.json?v=3.25.0)"><meta name="theme-color" content="#0074e5"><meta name="apple-mobile-web-app-capable" content="yes"><meta name="apple-mobile-web-app-status-bar-style" content="#0074e5"><meta name="apple-mobile-web-app-title" content="Longdo Map"><link rel="apple-touch-icon" href="[/main/img/icons/map/apple-touch-icon-152x152.png?v=3.25.0](https://map.longdo.com/main/img/icons/map/apple-touch-icon-152x152.png?v=3.25.0)"><link rel="mask-icon" href="[/main/img/icons/map/safari-pinned-tab.svg?v=3.25.0](https://map.longdo.com/main/img/icons/map/safari-pinned-tab.svg?v=3.25.0)" color="#0074e5"><meta name="msapplication-TileImage" content="/main/img/icons/map/msapplication-icon-144x144.png?v=3.25.0"><meta name="msapplication-TileColor" content="#0074e5"></head><body style="padding: 0px;
      margin: 0px;
      height: 100%;
      transition: background-color 0.3s ease 0s;
      background-color: #0074e5;"><img id="splash-screen-logo" src="[//map.longdo.com/themes/longdo/logo-nopadding2.png?version=3.25.0](https://map.longdo.com/themes/longdo/logo-nopadding2.png?version=3.25.0)" srcset="[//map.longdo.com/themes/longdo/logo-nopadding.png?version=3.25.0](https://map.longdo.com/themes/longdo/logo-nopadding.png?version=3.25.0),[ //map.longdo.com/themes/longdo/logo-nopadding2.png?version=3.25.0 2x](https://map.longdo.com/themes/longdo/logo-nopadding2.png?version=3.25.0)" style="z-index: 1;
        position: fixed;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        transition: opacity 0.15s ease 0s, top 0.15s ease 0s;
        height: 40px;"><div id="splash-screen-bg"></div><img id="loading-initial" src="[/main/img/loading.gif?version=3.25.0](https://map.longdo.com/main/img/loading.gif?version=3.25.0)" style="display: none;
        z-index: 1;
        position: fixed;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);"><div id="app"></div><script>function checkHideSplashScreen() {
        const params = new URLSearchParams(window.location.search);
        const platform = params.get('platform');
        const isHideSplashScreen = ['android', 'ios', 'desktop'].includes(
          platform
        );

        if (isHideSplashScreen) {
          document.querySelector('#loading-initial').style.display = 'unset';
          document.querySelector('body').style.backgroundColor = '#0074e5';
          document.querySelector('#splash-screen-logo').style.opacity = '0';
          document.querySelector('#splash-screen-logo').style.zIndex = '-99';
          document.querySelector('#splash-screen-bg').style.opacity = '0';
          document.querySelector('#splash-screen-bg').style.zIndex = '-99';
          document.querySelector('#splash-screen-bg').style.animation = 'unset';
        }
      }
      checkHideSplashScreen();</script><script src="[/main/js/chunk-vendors.f0b7eac9.js](https://map.longdo.com/main/js/chunk-vendors.f0b7eac9.js)"></script><script src="[/main/js/index.acc142e7.js](https://map.longdo.com/main/js/index.acc142e7.js)"></script><script type="module" src="https://static.cloudflareinsights.com/beacon.min.js/v31edd6df95cf4e85bb4c19e7a9bdbcba1788362987495" integrity="sha512-iIg7k2xntmwu6/uSb5tpc/hySgZc4eoL31yB29W6tJFo2akwjPWcEqnCEdJvGexCL0KEQwVYv5BlowfhVz26hg==" data-cf-beacon='{"version":"2024.11.0","token":"a48df47193f34a06b2ad4f94fbb402e7","spa":2}' crossorigin="anonymous"></script>
</body></html>
