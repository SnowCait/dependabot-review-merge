# dependabot-review-merge

A Codex skill that reviews Dependabot pull requests and merges safe updates sequentially.

## Installation

Clone the repository and link it to the Codex skills directory:

```sh
git clone https://github.com/SnowCait/dependabot-review-merge.git
mkdir -p ~/.agents/skills
ln -s "$(pwd)/dependabot-review-merge" ~/.agents/skills/dependabot-review-merge
```

Restart Codex if the skill is not detected.

## Usage

Invoke the skill explicitly with `$dependabot-review-merge`.

**Specific PRs:**

```text
$dependabot-review-merge Review and merge PRs #42 and #57 in OWNER/REPO, in that order.
```

**All Dependabot PRs:**

```text
$dependabot-review-merge Review and merge all open Dependabot PRs in OWNER/REPO.
```

**Filtered PRs:**

```text
$dependabot-review-merge Review and merge Patch and Minor updates in OWNER/REPO, excluding GitHub Actions.
```

PR numbers and filtering conditions can also be combined.

If no repository is specified, the current Git repository is used. If no PR selection is specified, all open Dependabot PRs are selected.

The skill stops if an update cannot be safely merged.
