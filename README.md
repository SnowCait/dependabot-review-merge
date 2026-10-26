# dependabot-review-merge

A repository-independent Codex Skill for substantively reviewing Dependabot pull requests and merging verified-safe updates one at a time. It works across languages, dependency ecosystems, and package managers.

The skill is **explicit invocation only**: `agents/openai.yaml` sets `policy.allow_implicit_invocation: false`.

## Usage

Invoke the installed skill in Codex and describe the repository and selection. If the repository is omitted, the current Git repository is used. No conditions means all open, unmerged Dependabot PRs.

PR numbers or URLs:

```text
$dependabot-review-merge Review and merge PRs #42 and #57 in OWNER/REPO, in that order.
```

```text
$dependabot-review-merge Review and merge https://github.com/OWNER/REPO/pull/42.
```

All open Dependabot PRs:

```text
$dependabot-review-merge Review and merge all open Dependabot PRs in OWNER/REPO.
```

Conditions and exclusions:

```text
$dependabot-review-merge In OWNER/REPO, review and merge Minor or Patch npm updates under /web with the dependencies label. Exclude Major updates and packages matching @vendor/*.
```

Numbers combined with conditions:

```text
$dependabot-review-merge Review PRs #42, #57 and #63 in OWNER/REPO. Merge only Patch updates to pytest under /tools with passing CI, in the listed order.
```

By default, numbered PRs restrict the candidates and conditions filter those candidates. State a union explicitly if that is intended. Conditions can combine package names, Major/Minor/Patch levels, ecosystems, directories, labels and CI status. Dependency conditions apply to Dependabot's intended update targets, whether direct or transitive, rather than incidental transitive changes. A group with any update target outside the conditions is excluded as a whole. All incidental changes still undergo safety review; ambiguity about which dependencies are targeted stops the run if it affects scope. Passing CI as a selection condition is not safety approval.

A narrower request such as “review only; do not merge” is respected.

## What happens

1. Resolve the repository and freeze the logical target set and processing order.
2. Review actual dependency/lockfile changes, upstream official sources, repository usage, compatibility, security, CI and necessary project-specific validation.
3. Resolve the repository's default merge method and protections, then merge only the reviewed HEAD using a server-side SHA guard. A required merge queue is observed until actual completion; auto-merge is never scheduled.
4. After each merge, wait two minutes and reconcile remaining targets, tracking rebases and uniquely identifiable regenerated successors. New unrelated PRs are not added.

Any selected update that is unsafe, failing, unmergeable or indeterminate stops the entire run. Later PRs are not processed past that failure. The report lists completed merges, the reason for stopping and all unprocessed targets. CI, merge-queue and active Dependabot waits each default to at most 20 minutes per logical target, with two-minute checks; retries do not reset these clocks. A user can specify a different budget.

The repository must have an identifiable default merge method and verifiable rules. A viewer's last-used merge preference is not sufficient evidence of a repository default. The skill does not change repository settings, update PR branches, post approvals or repair failing updates unless separately authorized.

## Installation

Requirements: Codex with local skill support, authenticated GitHub read/merge access, and tools needed for the target project's checks. GitHub CLI or an equivalent connector must support atomic expected-HEAD checks.

Clone this repository, then link its root into the user skills directory. For the requested local layout:

```sh
git clone https://github.com/SnowCait/dependabot-review-merge.git /Users/snowflake/workspace/dependabot-review-merge
mkdir -p ~/.agents/skills
ln -s /Users/snowflake/workspace/dependabot-review-merge ~/.agents/skills/dependabot-review-merge
```

Do not overwrite an existing checkout, file or link. The `ln` command intentionally omits overwrite flags. Until the implementation PR is merged, check out its implementation branch in the clone. The symlink follows the checkout, so keep it on a revision containing the skill files. If Codex does not discover the skill immediately, restart it.

## Files

```text
SKILL.md             Review, selection, merge and reconciliation instructions
agents/openai.yaml   UI metadata and explicit-invocation policy
README.md            Usage and installation
LICENSE              MIT License
```

No scripts or test framework are required. Skill validation and offline scenario checks do not review or mutate live Dependabot PRs.

The format, user skills location, symlink discovery and invocation policy follow the [official Codex Skill documentation](https://learn.chatgpt.com/docs/build-skills).

## License

[MIT License](LICENSE). Copyright (c) 2026 SnowCait.
