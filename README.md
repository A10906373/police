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
{{Short description|HTTP extension supporting TLS encryption}}
{{pp-pc}}
{{Use dmy dates|date=November 2021|cs1-dates=yy}}
{{HTTP}}
{{IPstack}}

'''Hypertext Transfer Protocol Secure''' ('''HTTPS''') is an extension of the [[HTTP|Hypertext Transfer Protocol]] (HTTP). It uses [[encryption]] for [[Secure communications|secure communication]] over a [[computer network]], and is widely used on the [[Internet]].<ref>{{cite web |url=https://support.google.com/webmasters/answer/6073543?hl=en |publisher=Google Inc. |work=Google Support |access-date=20 October 2018 |title=Secure your site with HTTPS |archive-url=https://web.archive.org/web/20150301023624/https://support.google.com/webmasters/answer/6073543?hl=en |archive-date=1 March 2015 |url-status=live }}</ref><ref>{{cite web |url=https://www.instantssl.com/ssl-certificate-products/https.html |publisher=[[Xcitium|Comodo CA Limited]] |quote=Hyper Text Transfer Protocol Secure (HTTPS) is the secure version of HTTP [...] |access-date=20 October 2018 |title=What is HTTPS? |url-status=unfit |archive-url=https://web.archive.org/web/20150212105201/https://www.instantssl.com/ssl-certificate-products/https.html |archive-date=12 February 2015 }}</ref> In HTTPS, the [[communication protocol]] is encrypted using [[Transport Layer Security]] (TLS) or, formerly, [[Secure Sockets Layer]] (SSL). The protocol is therefore also referred to as '''HTTP over TLS''',<ref>{{cite IETF |title=HTTP Semantics |rfc=9110 |sectionname=https URI Scheme |section=4.2.2 |date=June 2022 |publisher=[[Internet Engineering Task Force|IETF]]}}</ref> or '''HTTP over SSL'''.

The principal motivations for HTTPS are [[authentication]] of the accessed [[website]] and protection of the [[Information privacy|privacy]] and [[Data integrity|integrity]] of the exchanged data while it is in transit. It protects against [[man-in-the-middle attack]]s, and the bidirectional [[Block cipher mode of operation|block cipher encryption]] of communications between a [[Client (computing)|client]] and [[Server (computing)|server]] protects the communications against [[eavesdropping]] and [[Tamper-evident#Tampering|tampering]].<ref name=httpse>{{cite web |url=https://www.eff.org/https-everywhere/faq |title=HTTPS Everywhere FAQ |access-date=20 October 2018 |archive-url=https://web.archive.org/web/20181114011956/https://www.eff.org/https-everywhere/faq/ |archive-date=14 November 2018 |url-status=live |date=8 November 2016}}</ref><ref>{{Cite web|url=https://w3techs.com/technologies/details/ce-httpsdefault/all/all|title=Usage Statistics of Default protocol https for Websites, July 2019|website=w3techs.com|access-date=20 July 2019|archive-url=https://web.archive.org/web/20190801134536/https://w3techs.com/technologies/details/ce-httpsdefault/all/all|archive-date=1 August 2019|url-status=live}}</ref> The authentication aspect of HTTPS requires a trusted third party to sign server-side [[Public key certificate|digital certificates]]. This was historically an expensive operation, which meant fully authenticated HTTPS connections were usually found only on secured payment transaction services and other secured corporate information systems on the [[World Wide Web]]. In 2016, a campaign by the [[Electronic Frontier Foundation]] with the support of [[web browser]] developers led to the protocol becoming more prevalent.<ref>{{Cite web |last=Reitman |first=Rainey |date=2016-12-22 |title=Defending the Digital Future: 2016 in Review |url=https://www.eff.org/2016-year-review-digital-rights |access-date=2026-09-12 |website=Electronic Frontier Foundation |language=en}}</ref> HTTPS has since 2018<ref>{{Cite web|url=https://www.welivesecurity.com/2018/09/03/majority-worlds-top-websites-https/|title=Majority of the world's top million websites now use HTTPS|website=welivesecurity.com|access-date=22 May 2025}}</ref> been used more often by web users than non-secure HTTP, primarily to protect page authenticity on all types of websites, secure accounts, and keep user communications, identity, and web browsing private.

==Overview==
{{Further|topic=|Transport Layer Security|||category=}}
[[File:Internet2.svg|thumb|[[URL]] beginning with the HTTPS scheme and the [[World Wide Web|WWW]] domain name label]]

The [[Uniform Resource Identifier]] (URI) scheme ''HTTPS'' has identical usage syntax to the HTTP scheme. However, HTTPS signals the browser to use an added encryption layer of SSL/TLS to protect the traffic. SSL/TLS is especially suited for HTTP, since it can provide some protection even if only one side of the communication is [[authentication|authenticated]]. This is the case with HTTP transactions over the Internet, where typically only the [[Web server|server]] is authenticated (by the client examining the server's [[public key certificate|certificate]]).

HTTPS creates a secure channel over an insecure network. This ensures reasonable protection from [[eavesdropping|eavesdroppers]] and [[man-in-the-middle attack]]s, provided that adequate [[cipher suite]]s are used and that the server certificate is verified and trusted.

Because HTTPS piggybacks HTTP entirely on top of TLS, the entirety of the underlying HTTP protocol can be encrypted. This includes the request's [[URL]], query parameters, headers, and cookies (which often contain identifying information about the user). However, because website addresses and [[Port (computer networking)|port]] numbers are necessarily part of the underlying [[TCP/IP]] protocols, HTTPS cannot protect their disclosure. In practice this means that even on a correctly configured web server, eavesdroppers can infer the IP address and port number of the web server, and sometimes even the domain name (e.g. www.example.org, but not the rest of the URL) that a user is communicating with, along with the amount of data transferred and the duration of the communication, though not the content of the communication.<ref name=httpse/>

Web browsers know how to trust HTTPS websites based on [[Certificate authority|certificate authorities]] that come pre-installed in their software. Certificate authorities are in this way being trusted by web browser creators to provide valid certificates. Therefore, a user should trust an HTTPS connection to a website when all of the following are true:

* The user trusts that their device, hosting the browser and the method to get the browser itself, is not compromised (i.e. there is no [[supply chain attack]]).
* The user trusts that the browser software correctly implements HTTPS with correctly pre-installed certificate authorities.
* The user trusts the certificate authority to vouch only for legitimate websites (i.e. the certificate authority is not compromised and there is no mis-issuance of certificates).
* The website provides a valid certificate, which means it was signed by a trusted authority.
* The certificate correctly identifies the website (e.g., when the browser visits "https://example.com", the received certificate is properly for "example.com" and not some other entity).
* The user trusts that the protocol's encryption layer (SSL/TLS) is sufficiently secure against eavesdroppers.

HTTPS is especially important over insecure networks and networks that may be subject to tampering. Insecure networks, such as public [[Wi-Fi]] access points, allow anyone on the same local network to [[packet analyzer|packet-sniff]] and discover sensitive information not protected by HTTPS. Additionally, some free-to-use and paid [[wireless LAN|WLAN]] networks have been observed tampering with webpages by engaging in [[packet injection]] in order to serve their own ads on other websites. This practice can be exploited maliciously in many ways, such as by injecting [[malware]] onto webpages and stealing users' private information.<ref>{{cite web |title=Hotel Wifi JavaScript Injection |url=https://justinsomnia.org/2012/04/hotel-wifi-javascript-injection/ |date=3 April 2012 |work=JustInsomnia |access-date=20 October 2018 |archive-url=https://web.archive.org/web/20181118154608/https://justinsomnia.org/2012/04/hotel-wifi-javascript-injection/ |archive-date=18 November 2018 |url-status=live }}</ref>

HTTPS is also important for connections over the [[Tor (network)|Tor network]], as malicious Tor nodes could otherwise damage or alter the contents passing through them in an insecure fashion and inject malware into the connection. This is one reason why the [[Electronic Frontier Foundation]] and [[the Tor Project]] started the development of [[HTTPS Everywhere]],<ref name=httpse/> which is included in Tor Browser.<ref>{{cite web |url=https://www.torproject.org/projects/torbrowser.html.en |title=What is Tor Browser? |author=The Tor Project, Inc. |work=TorProject.org |access-date=30 May 2012 |archive-url=https://archive.today/20130717222709/https://www.torproject.org/projects/torbrowser.html.en |archive-date=17 July 2013 |url-status=live }}</ref>

As more information is revealed about global [[mass surveillance]] and criminals stealing personal information, the use of HTTPS security on all websites is becoming increasingly important regardless of the type of Internet connection being used.<ref>{{cite web |url=https://open.blogs.nytimes.com/2014/11/13/embracing-https/ |title=Embracing HTTPS |work=The New York Times |date=13 November 2014 |access-date=20 October 2018 |last1=Konigsburg |first1=Eitan |last2=Pant |first2=Rajiv |last3=Kvochko |first3=Elena |archive-url=https://web.archive.org/web/20190108190000/https://open.blogs.nytimes.com/2014/11/13/embracing-https/ |archive-date=8 January 2019 |url-status=live }}</ref><ref>{{cite web |url=https://freedom.press/news-advocacy/fifteen-months-after-the-nsa-revelations-why-arenat-more-news-organizations-using-https/ |title=Fifteen Months After the NSA Revelations, Why Aren't More News Organizations Using HTTPS? |publisher=Freedom of the Press Foundation |date=12 September 2014 |access-date=20 October 2018 |last=Gallagher |first=Kevin |archive-url=https://web.archive.org/web/20180810204919/https://freedom.press/news-advocacy/fifteen-months-after-the-nsa-revelations-why-arenat-more-news-organizations-using-https/ |archive-date=10 August 2018 |url-status=live }}</ref> Even though [[metadata]] about individual pages that a user visits might not be considered sensitive, when aggregated it can reveal a lot about the user and compromise the user's privacy.<ref>{{cite web |url=https://webmasters.googleblog.com/2014/08/https-as-ranking-signal.html |title=HTTPS as a ranking signal |date=6 August 2014 |publisher=Google Inc. |quote=You can make your site secure with HTTPS (Hypertext Transfer Protocol Secure) [...] |work=Google Webmaster Central Blog |access-date=20 October 2018 |archive-url=https://web.archive.org/web/20181017052432/https://webmasters.googleblog.com/2014/08/https-as-ranking-signal.html |archive-date=17 October 2018 |url-status=live }}</ref><ref>{{cite web |url=https://www.youtube.com/watch?v=cBhZ6S0PFCY |title=Google I/O 2014 - HTTPS Everywhere |publisher=Google Developers |date=26 June 2014 |access-date=20 October 2018 |last1=Grigorik |first1=Ilya |last2=Far |first2=Pierre |archive-url=https://web.archive.org/web/20181120144918/https://www.youtube.com/watch?v=cBhZ6S0PFCY |archive-date=20 November 2018 |url-status=live }}</ref><ref name=deployhttpscorrectly/>

Deploying HTTPS also allows the use of [[HTTP/2]] and [[HTTP/3]] (and their predecessors [[SPDY]] and [[QUIC]]), which are new HTTP versions designed to reduce page load times, size, and latency.

It is recommended to use [[HTTP Strict Transport Security]] (HSTS) with HTTPS to protect users from man-in-the-middle attacks, especially [[Moxie Marlinspike#SSL stripping|SSL stripping]].<ref name=deployhttpscorrectly>{{cite web |title=How to Deploy HTTPS Correctly |url=https://www.eff.org/https-everywhere/deploying-https |access-date=20 October 2018 |archive-url=https://web.archive.org/web/20181010233702/https://www.eff.org/https-everywhere/deploying-https |archive-date=10 October 2018 |url-status=live |date=15 November 2010 }}</ref><ref>{{cite web |url=https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security |title=HTTP Strict Transport Security |work=Mozilla Developer Network |access-date=20 October 2018 |archive-url=https://web.archive.org/web/20181019171534/https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security |archive-date=19 October 2018 |url-status=live }}</ref>

HTTPS should not be confused with the seldom-used [[Secure Hypertext Transfer Protocol|Secure HTTP]] (S-HTTP) specified in RFC 2660.

===Usage in websites===
{{As of|2018|04}}, 33.2%<!-- percentages not show; calculated from 331889 sites stated--> of Alexa top 1,000,000 websites use HTTPS as default<ref>{{cite web |url=https://statoperator.com/research/https-usage-statistics-on-top-websites/ |title=HTTPS usage statistics on top 1M websites |website=StatOperator.com |access-date=20 October 2018 |archive-url=https://web.archive.org/web/20190209055130/https://statoperator.com/research/https-usage-statistics-on-top-websites/ |archive-date=9 February 2019 |url-status=live }}</ref> and 70%<!-- 78.68% for the US only--> of page loads (measured by Firefox Telemetry) use HTTPS.<ref>{{cite web |url=https://letsencrypt.org/stats/ |title=Let's Encrypt Stats |website=LetsEncrypt.org |access-date=20 October 2018 |archive-url=https://web.archive.org/web/20181019221028/https://letsencrypt.org/stats/ |archive-date=19 October 2018 |url-status=live }}</ref> {{As of|2025|06}}, 71.2% of the Internet's <!-- "across 150,000 SSL- and TLS-enabled websites" likely right number, not 134,380 --> 150,000 most popular websites have a secure implementation of HTTPS (up from 58.4%<!-- then of 135,422 sites, also wrong number? --> in December 2022),<ref>{{cite web |url=https://www.ssllabs.com/ssl-pulse/ |title=Qualys SSL Labs - SSL Pulse |website=www.ssllabs.com |date=2 June 2025 |access-date=7 December 2022 |archive-url=https://web.archive.org/web/20221207004823/https://www.ssllabs.com/ssl-pulse/ |archive-date=7 December 2022 |url-status=live }}.</ref> However, despite [[TLS 1.3]]'s release in 2018, adoption has been slow, with many still remaining on the older TLS 1.2 protocol.<ref>{{Cite web |date=2020-04-06 |title=TLS 1.3: Slow adoption of stronger web encryption is empowering the bad guys |url=https://www.helpnetsecurity.com/2020/04/06/tls-1-3-adoption/ |access-date=2022-05-23 |website=Help Net Security |language=en-US |archive-date=24 May 2022 |archive-url=https://web.archive.org/web/20220524002257/https://www.helpnetsecurity.com/2020/04/06/tls-1-3-adoption/ |url-status=live }}</ref>

===Browser integration===

Most [[Web browser|browsers]] display a warning if they receive an invalid certificate. Older browsers, when connecting to a site with an invalid certificate, would present the user with a [[dialog box]] asking whether they wanted to continue. Newer browsers display a warning across the entire window. Newer browsers also prominently display the site's security information in the [[address bar]]. [[Extended validation certificate]]s show the legal entity on the certificate information. Most browsers also display a warning to the user when visiting a site that contains a mixture of encrypted and unencrypted content. Additionally, many [[Content-control software|web filters]] return a security warning when visiting prohibited websites.
{{gallery
|title=Comparison between different kinds of [[Transport Layer Security|SSL/TLS]] certificates<br><small>(Using [[Firefox]] as an example)</small>
|height=170
|width=300
|align=center
|File:Extended Validation on Firefox 133 screenshot.webp|Many web browsers, including Firefox (shown here), use the [[address bar]] to tell the user that their connection is secure, an [[Extended Validation Certificate]] should identify the legal entity for the certificate.
|File:HTTPS on Firefox 133 screenshot.webp|When accessing a site only with a common certificate, on the address bar of [[Firefox]] and other [[Web browser|browser]]s, a "lock" sign appears.
|File:Self-signed certificate warning on Firefox 133 screenshot.webp|Most web browsers alert the user when visiting sites that have invalid security certificates.
}}

The [[Electronic Frontier Foundation]], opining that "In an ideal world, every web request could be defaulted to HTTPS", has provided an add-on called HTTPS Everywhere for [[Mozilla Firefox]], [[Google Chrome]], [[Chromium (web browser)|Chromium]], and [[Android (operating system)|Android]], which enables HTTPS by default for hundreds of frequently used websites.<ref>{{cite web |first=Peter |last=Eckersley |url=https://www.eff.org/deeplinks/2010/06/encrypt-web-https-everywhere-firefox-extension |title=Encrypt the Web with the HTTPS Everywhere Firefox Extension |work=EFF blog |date=17 June 2010 |access-date=20 October 2018 |archive-url=https://web.archive.org/web/20181125102636/https://www.eff.org/deeplinks/2010/06/encrypt-web-https-everywhere-firefox-extension |archive-date=25 November 2018 |url-status=live }}</ref><ref>{{cite web |url=https://www.eff.org/https-everywhere |title=HTTPS Everywhere. |work=EFF projects |access-date=20 October 2018 |archive-url=https://web.archive.org/web/20110605022218/https://www.eff.org/https-everywhere |archive-date=5 June 2011 |url-status=live |date=7 October 2011 }}</ref>

Forcing a web browser to load only HTTPS content has been supported in Firefox starting in version 83.<ref>{{Cite web|title=HTTPS-Only Mode in Firefox|url=https://support.mozilla.org/en-US/kb/https-only-prefs|url-status=live|access-date=12 November 2021|archive-date=12 November 2021|archive-url=https://web.archive.org/web/20211112222245/https://support.mozilla.org/en-US/kb/https-only-prefs}}</ref> Starting in version 94, Google Chrome is able to "always use secure connections" if toggled in the browser's settings.<ref>{{Cite web |title=Manage Chrome safety and security - Android - Google Chrome Help |url=https://support.google.com/chrome/answer/10468685?hl=en&co=GENIE.Platform=Android |access-date=2022-03-07 |website=support.google.com |archive-date=7 March 2022 |archive-url=https://web.archive.org/web/20220307190622/https://support.google.com/chrome/answer/10468685?hl=en&co=GENIE.Platform=Android |url-status=live }}</ref><ref>{{Cite web |date=2021-07-19 |title=Hands on Chrome's HTTPS-First Mode |url=https://techdows.com/2021/07/hands-on-chromes-https-first-mode.html |access-date=2022-03-07 |website=Techdows |language=en-US |archive-date=7 March 2022 |archive-url=https://web.archive.org/web/20220307190617/https://techdows.com/2021/07/hands-on-chromes-https-first-mode.html |url-status=live |author1=Eswarlu |first=Venkat}}</ref> Prior to version 117, Google Chrome displayed a lock icon in the [[address bar]], which has since been replaced by a "tune" icon.<ref>{{cite web |last1=Crane |first1=Casey |title=Google to Replace the Padlock Icon in Chrome Version 117 |url=https://www.thesslstore.com/blog/google-to-replace-the-padlock-icon-in-chrome-version-117/#:%7E:text=But%20that's%20about%20to%20change,to%20have%20HTTPS%20by%20default. |website=Hashed Out |publisher=The SSL Store |access-date=5 May 2026}}</ref> Many users believe that a lock icon implies a website is safe, ignoring other security concerns.<ref>{{cite journal |last1=Ruoti |first1=Scott |last2=Monson |first2=Tyler |last3=Wu |first3=Jusin |last4=Zappala |first4=Daniel |last5=Seamons |first5=Kent |title=Weighing context and trade-offs: how suburban adults selected their online security posture |journal=Thirteenth Symposium on Usable Privacy and Security (SOUPS 2017) |date=12 July 2017 |pages=211-228 |url=https://dl.acm.org/doi/10.5555/3235924.3235942 |access-date=5 May 2026}}</ref>

==Security==
{{Main|Transport Layer Security#Security}}
The security of HTTPS is that of the underlying TLS, which typically uses long-term [[Public-key cryptography|public]] and private keys to generate a short-term [[session key]], which is then used to encrypt the data flow between the client and the server. [[X.509]] certificates are used to authenticate the server (and sometimes the client as well). As a consequence, [[certificate authority|certificate authorities]] and [[public key certificate]]s are necessary to verify the relation between the certificate and its owner, as well as to generate, sign, and administer the validity of certificates. While this can be more beneficial than verifying the identities via a [[web of trust]], the [[2013 mass surveillance disclosures]] drew attention to certificate authorities as a potential weak point allowing [[man-in-the-middle attack]]s.<ref>{{cite magazine |url=https://www.wired.com/2010/03/packet-forensics/ |title=Law Enforcement Appliance Subverts SSL |magazine=Wired |date=24 March 2010 |first=Ryan |last=Singel |access-date=20 October 2018 |archive-url=https://web.archive.org/web/20190117142906/https://www.wired.com/2010/03/packet-forensics/ |archive-date=17 January 2019 |url-status=live }}</ref><ref>{{cite web |url=https://www.eff.org/deeplinks/2010/03/researchers-reveal-likelihood-governments-fake-ssl |title=New Research Suggests That Governments May Fake SSL Certificates |first=Seth |last=Schoen |work=EFF |date=24 March 2010 |access-date=20 October 2018 |archive-url=https://web.archive.org/web/20160104234608/https://www.eff.org/deeplinks/2010/03/researchers-reveal-likelihood-governments-fake-ssl |archive-date=4 January 2016 |url-status=live }}</ref> An important property in this context is [[forward secrecy]], which ensures that encrypted communications recorded in the past cannot be retrieved and decrypted should long-term secret keys or passwords be compromised in the future. Not all web servers provide forward secrecy.<ref name=ecdhe>{{cite web |url=https://news.netcraft.com/archives/2013/06/25/ssl-intercepted-today-decrypted-tomorrow.html |title=SSL: Intercepted today, decrypted tomorrow |work=Netcraft |date=25 June 2013 |first=Robert |last=Duncan |access-date=20 October 2018 |archive-url=https://web.archive.org/web/20181006021916/https://news.netcraft.com/archives/2013/06/25/ssl-intercepted-today-decrypted-tomorrow.html |archive-date=6 October 2018 |url-status=live }}</ref>{{Update inline|reason=Does this still hold in 2015?|date=February 2015}}

For HTTPS to be effective, a site must be completely hosted over HTTPS.  If some of the site's contents are loaded over HTTP (scripts or images, for example), or if only a certain page that contains sensitive information, such as a log-in page, is loaded over HTTPS while the rest of the site is loaded over plain HTTP, the user will be vulnerable to attacks and surveillance. Additionally, [[HTTP cookie|cookies]] on a site served through HTTPS must have the [[secure cookie|secure attribute]] enabled.  On a site that has sensitive information on it, the user and the session will get exposed every time that site is accessed with HTTP instead of HTTPS.<ref name=deployhttpscorrectly/>

==Technical==

===Difference from HTTP===
HTTPS [[URL]]s begin with "https://" and use [[List of TCP and UDP port numbers|port]] 443 by default, whereas [[HTTP]] URLs begin with "http://" and use port 80 by default.

HTTP is not encrypted and thus is vulnerable to [[man-in-the-middle]] and [[eavesdropping attack]]s, which can let attackers gain access to website accounts and sensitive information, and modify webpages to inject [[malware]] or advertisements. HTTPS is designed to withstand such attacks and is considered secure against them (with the exception of HTTPS implementations that use deprecated versions of SSL).

===Network layers===
HTTP operates at the highest layer of the [[TCP/IP model]]—the [[application layer]]; as does the [[Transport Layer Security|TLS]] security protocol (operating as a lower sublayer of the same layer), which encrypts an HTTP message prior to transmission and decrypts a message upon arrival. Strictly speaking, HTTPS is not a separate protocol, but refers to the use of ordinary [[HTTP]] over an [[encryption|encrypted]] SSL/TLS connection.

HTTPS encrypts all message contents, including the HTTP headers and the request/response data. With the exception of the possible [[Chosen-ciphertext attack|CCA]] cryptographic attack described in the [[#Limitations|limitations]] section below, an attacker should at most be able to discover that a connection is taking place between two parties, along with their domain names and IP addresses.

===Server setup===
To prepare a web server to accept HTTPS connections, the administrator must create a [[public key certificate]] for the web server. This certificate must be signed by a trusted [[certificate authority]] for the web browser to accept it without warning. The authority certifies that the certificate holder is the operator of the web server that presents it. Web browsers are generally distributed with a list of [[root certificate|signing certificates of major certificate authorities]] so that they can verify certificates signed by them.

====Acquiring certificates====
A number of commercial [[Certificate authority|certificate authorities]] exist, offering paid-for SSL/TLS certificates of a number of types, including [[Extended Validation Certificate]]s.

[[Let's Encrypt]], launched in April 2016,<ref name="softpedia-launch">{{cite web |url=https://news.softpedia.com/news/let-s-encrypt-launched-today-currently-protects-3-8-million-domains-502857.shtml |title=Let's Encrypt Launched Today, Currently Protects 3.8 Million Domains |publisher=Softpedia News |first=Catalin |last=Cimpanu |date=12 April 2016 |access-date=20 October 2018 |archive-url=https://web.archive.org/web/20190209055129/https://news.softpedia.com/news/let-s-encrypt-launched-today-currently-protects-3-8-million-domains-502857.shtml |archive-date=9 February 2019 |url-status=live }}</ref> provides free and automated service that delivers basic SSL/TLS certificates to websites.<ref>{{cite web |url=http://www.eweek.com/security/let-s-encrypt-effort-aims-to-improve-internet-security |title=Let's Encrypt Effort Aims to Improve Internet Security |publisher=Quinstreet Enterprise |website=eWeek.com |date=18 November 2014 |access-date=20 October 2018 |last=Kerner |first=Sean Michael |archive-date=2 April 2023 |archive-url=https://web.archive.org/web/20230402154948/https://www.eweek.com/security/let-s-encrypt-effort-aims-to-improve-internet-security/ |url-status=live }}</ref> According to the [[Electronic Frontier Foundation]], Let's Encrypt will make switching from HTTP to HTTPS "as easy as issuing one command, or clicking one button."<ref>{{cite web |url=https://www.eff.org/deeplinks/2014/11/certificate-authority-encrypt-entire-web |title=Launching in 2015: A Certificate Authority to Encrypt the Entire Web |publisher=[[Electronic Frontier Foundation]] |date=18 November 2014 |access-date=20 October 2018 |last=Eckersley |first=Peter |archive-url=https://web.archive.org/web/20181118160126/https://www.eff.org/deeplinks/2014/11/certificate-authority-encrypt-entire-web |archive-date=18 November 2018 |url-status=live }}</ref> The majority of web hosts and cloud providers now leverage Let's Encrypt, providing free certificates to their customers.

====Use as access control====
The system can also be used for client [[authentication]] in order to limit access to a web server to authorized users. To do this, the site administrator typically creates a certificate for each user, which the user loads into their browser. Normally, the certificate contains the name and e-mail address of the authorized user and is automatically checked by the server on each connection to verify the user's identity, potentially without even requiring a password.

====In case of compromised secret (private) key====
An important property in this context is [[forward secrecy|perfect forward secrecy]] (PFS). Possessing one of the long-term asymmetric secret keys used to establish an HTTPS session should not make it easier to derive the short-term session key to then decrypt the conversation, even at a later time. [[Diffie–Hellman key exchange]] (DHE) and [[Elliptic-curve Diffie–Hellman]] key exchange (ECDHE) are in 2013 the only schemes known to have that property. In 2013, only 30% of Firefox, Opera, and Chromium Browser sessions used it, and nearly 0% of Apple's [[Safari (web browser)|Safari]] and [[Internet Explorer|Microsoft Internet Explorer]] sessions.<ref name=ecdhe/> TLS 1.3, published in August 2018, dropped support for ciphers without forward secrecy. {{As of|2019|02|df=US}}, 96.6% of web servers surveyed support some form of forward secrecy, and 52.1% will use forward secrecy with most browsers.<ref>{{cite web |author1=Qualys SSL Labs |title=SSL Pulse |url=https://www.ssllabs.com/ssl-pulse/ |access-date=25 February 2019 |archive-url=https://web.archive.org/web/20190215213454/https://www.ssllabs.com/ssl-pulse/ |archive-date=15 February 2019 |format=3 February 2019|author1-link=Qualys }}</ref> {{As of|2023|07|df=US}}, 99.6% of web servers surveyed support some form of forward secrecy, and 75.2% will use forward secrecy with most browsers.<ref>{{Cite web |title=Qualys SSL Labs - SSL Pulse |url=https://www.ssllabs.com/ssl-pulse/ |access-date=2023-09-04 |website=www.ssllabs.com}}</ref>

=====Certificate revocation=====
{{main|Certificate revocation}}
A certificate may be revoked before it expires, for example because the secrecy of the private key has been compromised. Newer versions of popular browsers such as [[Firefox]],<ref>{{cite web |url=https://www.mozilla.org/en-US/privacy/ |title=Mozilla Firefox Privacy Policy |publisher=[[Mozilla Foundation]] |date=27 April 2009 |access-date=20 October 2018 |archive-url=https://web.archive.org/web/20181018063732/https://www.mozilla.org/en-US/privacy/ |archive-date=18 October 2018 |url-status=live }}</ref> [[Opera (web browser)|Opera]],<ref>{{cite news |url=https://news.softpedia.com/news/Opera-8-launched-on-FTP-1330.shtml |title=Opera 8 launched on FTP |publisher=[[Softpedia]] |date=19 April 2005 |access-date=20 October 2018 |archive-url=https://web.archive.org/web/20190209055128/https://news.softpedia.com/news/Opera-8-launched-on-FTP-1330.shtml |archive-date=9 February 2019 |url-status=live }}</ref> and [[Internet Explorer]] on [[Windows Vista]]<ref>{{cite web |last=Lawrence |first=Eric |date=31 January 2006 |url=https://docs.microsoft.com/en-us/previous-versions/aa980989(v=msdn.10) |title=HTTPS Security Improvements in Internet Explorer 7 |website=[[Microsoft Docs]] |access-date=24 October 2021 |archive-date=24 October 2021 |archive-url=https://web.archive.org/web/20211024181937/https://docs.microsoft.com/en-us/previous-versions/aa980989(v=msdn.10) |url-status=live }}</ref> implement the [[Online Certificate Status Protocol]] (OCSP) to verify that this is not the case. The browser sends the certificate's serial number to the certificate authority or its delegate via OCSP (Online Certificate Status Protocol) and the authority responds, telling the browser whether the certificate is still valid or not.<ref>{{cite web |url=https://tools.ietf.org/html/rfc2560 |title=Online Certificate Status Protocol – OCSP |publisher=[[Internet Engineering Task Force]] |date=20 June 1999 |last1=Myers |first1=Michael |last2=Ankney |first2=Rich |last3=Malpani |first3=Ambarish |last4=Galperin |first4=Slava |last5=Adams |first5=Carlisle |doi=10.17487/RFC2560 |access-date=20 October 2018 |archive-url=https://web.archive.org/web/20110825095059/http://tools.ietf.org/html/rfc2560 |archive-date=25 August 2011 |url-status=live }}</ref> The CA may also issue a [[Certificate revocation list|CRL]] to tell people that these certificates are revoked. CRLs are no longer required by the CA/Browser forum,<ref>{{cite web |url=https://cabforum.org/baseline-requirements-documents/ |title=Baseline Requirements |date=4 September 2013 |publisher=CAB Forum |access-date=1 November 2021 |url-status=live |archive-date=20 October 2014 |archive-url=https://web.archive.org/web/20141020234802/https://cabforum.org/baseline-requirements-documents/ }}</ref>{{Update inline|date=April 2025|reason=CRLs are actually required.}} nevertheless, they are still widely used by the CAs. Most revocation statuses on the Internet disappear soon after the expiration of the certificates.<ref name=RS_1>{{cite book| author1=Korzhitskii, N.| author2=Carlsson, N.| title=Passive and Active Measurement| chapter=Revocation Statuses on the Internet| series=Lecture Notes in Computer Science| date=30 March 2021| volume=12671| pages=175–191| doi=10.1007/978-3-030-72582-2_11| arxiv=2102.04288| isbn=978-3-030-72581-5}}</ref>

===Limitations===
SSL (Secure Sockets Layer) and TLS (Transport Layer Security) encryption can be configured in two modes: ''simple'' and ''mutual''. In simple mode, authentication is only performed by the server. The mutual version requires the user to install a personal [[client certificate]] in the web browser for user authentication.<ref>{{cite web |url=https://support.google.com/chrome/a/answer/6080885?hl=en |title=Manage client certificates on Chrome devices – Chrome for business and education Help |website=support.google.com |access-date=2018-10-20 |archive-url=https://web.archive.org/web/20190209055127/https://support.google.com/chrome/a/answer/6080885?hl=en |archive-date=2019-02-09 |url-status=live }}</ref> In either case, the level of protection depends on the correctness of the [[implementation]] of the software and the [[cipher|cryptographic algorithms]] in use.<ref>{{Cite web |title=Practical Cryptography |url=https://www.schneier.com/books/practical-cryptography/ |access-date=2026-04-14 |website=Schneier on Security |language=en-US |isbn=0471223573}}</ref>

SSL/TLS does not prevent the indexing of the site by a [[web crawler]], and in some cases the [[Uniform resource identifier|URI]] of the encrypted resource can be inferred by knowing only the intercepted request/response size.<ref>{{cite web |url=https://www.exploit-db.com/docs/english/13026-the-pirate-bay-un-ssl.pdf |title=The Pirate Bay un-SSL |last=Pusep |first=Stanislaw |date=2008-07-31 |access-date=2018-10-20 |archive-url=https://web.archive.org/web/20180620001518/https://www.exploit-db.com/docs/english/13026-the-pirate-bay-un-ssl.pdf |archive-date=2018-06-20 |url-status=live }}</ref> This allows an attacker to have access to the [[plaintext]] (the publicly available static content), and the [[ciphertext|encrypted text]] (the encrypted version of the static content), permitting a [[Chosen-ciphertext attack|cryptographic attack]].<ref>{{Cite report |url=https://www.rfc-editor.org/info/rfc8446/ |title=The Transport Layer Security (TLS) Protocol Version 1.3 |last=Rescorla |first=Eric |date=August 2018 |publisher=Internet Engineering Task Force}}</ref><ref>{{Cite journal |last=Panchenko |first=Andriy |last2=Lanze |first2=Fabian |last3=Zinnen |first3=Andreas |last4=Henze |first4=Martin |last5=Pennekamp |first5=Jan |last6=Wehrle |first6=Klaus |last7=Engel |first7=Thomas |date=2016-02-23 |title=Website Fingerprinting at Internet Scale |url=https://www.researchgate.net/publication/306097739_Website_Fingerprinting_at_Internet_Scale |journal=Conference: Network and Distributed System Security Symposium |doi=10.14722/ndss.2016.23477|doi-access=free }}</ref>

Because [[Transport Layer Security|TLS]] operates at a protocol level below that of HTTP and has no knowledge of the higher-level protocols, TLS servers can only strictly present one certificate for a particular address and port combination.<ref>{{cite web |url=https://httpd.apache.org/docs/2.0/ssl/ssl_faq.html#vhosts |title=SSL/TLS Strong Encryption: FAQ |work=apache.org |access-date=2018-10-20 |archive-url=https://web.archive.org/web/20181019105423/http://httpd.apache.org/docs/2.0/ssl/ssl_faq.html#vhosts |archive-date=2018-10-19 |url-status=live }}</ref> In the past, this meant that it was not feasible to use [[Virtual hosting#Name-based|name-based virtual hosting]] with HTTPS. A solution called [[Server Name Indication]] (SNI) exists, which sends the hostname to the server before encrypting the connection, although older browsers do not support this extension. Support for SNI is available since [[Firefox]] 2, [[Opera (web browser)|Opera]] 8, [[Safari (web browser)|Apple Safari]] 2.1, [[Google Chrome]] 6, and [[Internet Explorer 7]] on [[Windows Vista]].<ref>{{cite web |url=https://blogs.msdn.microsoft.com/ie/2005/10/22/upcoming-https-improvements-in-internet-explorer-7-beta-2/ |title=Upcoming HTTPS Improvements in Internet Explorer 7 Beta 2 |last=Lawrence |first=Eric |publisher=[[Microsoft]] |date=2005-10-22 |access-date=2018-10-20 |archive-url=https://web.archive.org/web/20180920113838/https://blogs.msdn.microsoft.com/ie/2005/10/22/upcoming-https-improvements-in-internet-explorer-7-beta-2/ |archive-date=2018-09-20 |url-status=live }}</ref><ref>{{cite web |url=https://blog.ebrahim.org/2006/02/21/server-name-indication-sni/ |title=Server Name Indication (SNI) |work=inside aebrahim's head |date=2006-02-21 |access-date=2018-10-20 |archive-url=https://web.archive.org/web/20180810173628/https://blog.ebrahim.org/2006/02/21/server-name-indication-sni/ |archive-date=10 August 2018 |url-status=live }}</ref><ref>{{cite web |url=https://bugzilla.mozilla.org/show_bug.cgi?id=116169 |title=Browser support for TLS server name indication |access-date=2018-10-20 |last=Pierre |first=Julien |date=2001-12-19 |work=Bugzilla |publisher=Mozilla Foundation |archive-url=https://web.archive.org/web/20181008070112/https://bugzilla.mozilla.org/show_bug.cgi?id=116169 |archive-date=2018-10-08 |url-status=live }}</ref>

A sophisticated type of [[man-in-the-middle attack]] called SSL stripping was presented at the 2009 [[Black Hat Briefings|Blackhat Conference]]. This type of attack defeats the security provided by HTTPS by changing the {{code|https:}} link into an {{code|http:}} link, taking advantage of the fact that few Internet users actually type "https" into their browser interface: they get to a secure site by clicking on a link, and thus are fooled into thinking that they are using HTTPS when in fact they are using HTTP. The attacker then communicates in clear with the client.<ref>{{cite web |url=https://moxie.org/software/sslstrip/index.html |title=sslstrip 0.9 |access-date=20 October 2018 |archive-url=https://web.archive.org/web/20180620042059/https://moxie.org/software/sslstrip/index.html |archive-date=20 June 2018 |url-status=live }}</ref> This prompted the development of a countermeasure in HTTP called [[HTTP Strict Transport Security]].{{fact|date=April 2024}}

HTTPS has been shown to be vulnerable to a range of [[traffic analysis]] attacks. Traffic analysis attacks are a type of [[side-channel attack]] that relies on variations in the timing and size of traffic in order to infer properties about the encrypted traffic itself. Traffic analysis is possible because SSL/TLS encryption changes the contents of traffic, but has minimal impact on the size and timing of traffic. In May 2010, a research paper by researchers from [[Microsoft Research]] and [[Indiana University Bloomington|Indiana University]] discovered that detailed sensitive user data can be inferred from side channels such as packet sizes. The researchers found that, despite HTTPS protection in several high-profile, top-of-the-line web applications in healthcare, taxation, investment, and web search, an eavesdropper could infer the illnesses/medications/surgeries of the user, his/her family income, and investment secrets.<ref>{{cite journal |url=https://www.microsoft.com/en-us/research/publication/side-channel-leaks-in-web-applications-a-reality-today-a-challenge-tomorrow/ |title=Side-Channel Leaks in Web Applications: a Reality Today, a Challenge Tomorrow |journal=Microsoft Research |publisher=[[Institute of Electrical and Electronics Engineers|IEEE]] Symposium on Security & Privacy 2010 |date=2010-05-20 |author1=Shuo Chen |author2=Rui Wang |author3=XiaoFeng Wang |author4=Kehuan Zhang |access-date=2018-10-20 |archive-url=https://web.archive.org/web/20180722120329/https://www.microsoft.com/en-us/research/publication/side-channel-leaks-in-web-applications-a-reality-today-a-challenge-tomorrow/ |archive-date=22 July 2018 |url-status=live }}</ref>

The fact that most modern websites, including Google, Yahoo!, and Amazon, use HTTPS causes problems for many users trying to access public Wi-Fi hot spots, because a [[captive portal]] Wi-Fi hot spot login page fails to load if the user tries to open an HTTPS resource.<ref>{{cite web |first=Matthew |last=Guaay |url=https://zapier.com/blog/open-wifi-login-page/ |title=How to Force a Public Wi-Fi Network Login Page to Open |date=2017-09-21 |access-date=2018-10-20 |archive-url=https://web.archive.org/web/20180810143254/https://zapier.com/blog/open-wifi-login-page/ |archive-date=2018-08-10 |url-status=live }}</ref> Several websites, such as [http://nossl.sh NoSSL.sh], guarantee that they will always remain accessible by HTTP<ref>{{Cite web |title=nossl.sh HTTP-only disclaimer |url=http://nossl.sh/disclaimer |access-date=2025-12-18 |website=nossl.sh |language=en}}</ref>.

==History==
{{Main|Transport Layer Security#History and development}}
[[Netscape Communications]] created HTTPS in 1994 for its [[Netscape Navigator]] web browser.<ref>{{cite book |url=https://books.google.com/books?id=FLvsis4_QhEC&pg=PA344 |title=Embedded Software: The Works |last=Walls |first=Colin |year=2005 |pages=344 |isbn=0-7506-7954-9 |publisher=Newnes |access-date=20 October 2018 |archive-url=https://web.archive.org/web/20190209055124/https://www.google.com/books/edition/_/FLvsis4_QhEC?hl=en&gbpv=1&pg=PA344 |archive-date=9 February 2019 |url-status=live }}</ref> Originally, HTTPS was used with the [[Secure Sockets Layer|SSL]] protocol.<ref name=":0" /> The original SSL protocol was developed by [[Taher Elgamal]], chief scientist at [[Netscape|Netscape Communications]].<ref name="Messmer">{{cite news |last=Messmer |first=Ellen |title=Father of SSL, Dr. Taher Elgamal, Finds Fast-Moving IT Projects in the Middle East |url=http://www.networkworld.com/news/2012/120412-elgamal-264739.html |url-status=dead |archive-url=https://web.archive.org/web/20140531105537/http://www.networkworld.com/news/2012/120412-elgamal-264739.html |archive-date=31 May 2014 |access-date=30 May 2014 |work=Network World}}</ref><ref name="Greene">{{cite news |last=Greene |first=Tim |title=Father of SSL says despite attacks, the security linchpin has lots of life left |url=http://www.networkworld.com/news/2011/101111-elgamal-251806.html |url-status=dead |archive-url=https://web.archive.org/web/20140531105257/http://www.networkworld.com/news/2011/101111-elgamal-251806.html |archive-date=31 May 2014 |access-date=30 May 2014 |work=Network World}}</ref><ref name="Oppliger">{{cite book |last=Oppliger |first=Rolf |title=SSL and TLS: Theory and Practice |publisher=[[Artech House]] |year=2016 |isbn=978-1-60807-999-5 |edition=2nd |page=13 |chapter=Introduction |access-date=2018-03-01 |chapter-url=https://books.google.com/books?id=jm6uDgAAQBAJ&pg=PA15 |via=Google Books}}</ref> As SSL evolved into [[Transport Layer Security]] (TLS), HTTPS was formally specified by RFC 2818<ref>{{Cite report |url=https://datatracker.ietf.org/doc/html/rfc2818 |title=HTTP Over TLS |last=Rescorla |first=Eric |date=May 2000 |publisher=Internet Engineering Task Force |issue=RFC 2818}}</ref> in May 2000. Google announced in February 2018 that its Chrome browser would mark HTTP sites as "Not Secure" after July 2018.<ref name=":0">{{Cite web|url=https://blog.chromium.org/2018/02/a-secure-web-is-here-to-stay.html|title=A secure web is here to stay|website=Chromium Blog|archive-url=https://web.archive.org/web/20190424215132/https://blog.chromium.org/2018/02/a-secure-web-is-here-to-stay.html|archive-date=24 April 2019|access-date=22 April 2019|url-status=live}}</ref> This move was to encourage website owners to implement HTTPS, as an effort to make the [[World Wide Web]] more secure.

==See also==
* [[Transport Layer Security]]
* [[Bullrun (decryption program)]]{{snd}} a secret anti-encryption program run by the US [[National Security Agency]]
* [[Computer security]]
* [[HTTP Strict Transport Security|HSTS]]
* [[Opportunistic encryption]]
* [[Stunnel]]
* [[Bicycle attack]]

==References==
{{reflist}}

==External links==
{{commons category}}
*{{IETF RFC|8446|link=no}}: The Transport Layer Security (TLS) Protocol Version 1.3

{{Web interfaces}}
{{Web browsers|fsp}}
{{URI scheme}}
{{SSL/TLS}}
{{Internet censorship circumvention technologies}}

[[Category:Hypertext Transfer Protocol]]
[[Category:Cryptographic protocols]]
[[Category:Secure communication]]
[[Category:URI schemes]]
[[Category:Transport Layer Security]]
[[Category:Internet properties established in 1994]]
