# Project facts

Facts of the GitHub issue process of Watch Face Basic View (repository `Panaith/watchface_basic_view`) that `SKILL.md` refers to by name. The defaults are plain GitHub Issues; each section says what applies in this repository (checked 2026-10-23).

## Contents

- Work item types and states
- Developer Notes
- Release Notes
- Work started state
- Review state
- Pull request title and description
- Required fields
- Agent tooling parent
- Reviewers
- Review threads
- Work item fields

## Work item types and states

Types: a bug (label `bug`, or the issue type Bug) and an enhancement (label `enhancement`, or the issue type Feature), plus tasks if the project uses them. States: only open and closed (`stateReason` completed or not planned); anything finer is the Status of a GitHub Projects board, if the project has one. Fields to read for the analysis: the issue body (with the answers of an issue form, if the repository has one in `.github/ISSUE_TEMPLATE/`), the labels, the comments, and for a bug the steps to reproduce in the body.

In this repository (checked 2026-10-23): bugs and enhancements are told apart by the default labels `bug` and `enhancement`; no issue types, no issue forms or templates (`.github/ISSUE_TEMPLATE/` does not exist), and no GitHub Projects board, so an issue is only open or closed.

## Developer Notes

Place written in chapter 10: an issue comment whose first line is the heading `Developer Notes`, updated in place (`gh api --method PATCH repos/Panaith/watchface_basic_view/issues/comments/{comment id}`) instead of a new comment each time.

## Release Notes

Default: the project keeps no release notes per issue; skip step 2 of chapter 15.


## Work started state

What is done before the first change (chapter 13): assign yourself (`gh issue edit {n} --repo Panaith/watchface_basic_view --add-assignee @me`) and, if the project uses a GitHub Projects board, set its Status to In Progress (`gh project item-edit`; the token needs the `project` scope: `gh auth refresh -s project`, which is a change of the sign-in and needs approval). Skip what is already so.

In this repository: assign yourself; no board, no label.

## Review state

What is done when the PR is created (chapter 15): nothing on the issue itself by default (it stays open; the PR with `Closes #{n}` closes it on merge only when `master` is the repository's default branch, otherwise it is closed by hand after the merge, with approval); with a board, set its Status to In Review.

In this repository: no board.

## Pull request title and description

Default: title `Bug {id}: {issue title}` or `Feature {id}: {issue title}`, `{id}` the issue number written without `#`. Description sections: Problem (one sentence on why), Cause (bugs), Fix, Testing; the last line `Closes #{id}` (works only for a PR into the default branch; see `SKILL.md`, chapter 15) (the only `#` with a number: `SKILL.md`, chapter 15). The repository's `.github/pull_request_template.md`, if there is one, wins over these sections.

In this repository: PRs are squash-merged and the commits on `master` follow Conventional Commits, so the PR title is a Conventional Commits title scoped by app where useful, which becomes the squash commit's title: `feat(watchface): {issue title}` for an enhancement, `fix(watchface): {issue title}` for a bug, `docs: ...`, `chore(ai_tooling): ...`. Write the issue number only in the last line `Closes #{id}`. There is no `.github/pull_request_template.md`; the description sections above apply.

## Required fields

What the repository requires on an issue or a PR; a missing one is not rejected by GitHub itself, but by an issue form, a workflow or a ruleset.

- Required labels, assignee, milestone, issue form fields: none (no issue forms, workflows or rulesets).
- Creating a follow-up bug issue (found during the work): ask the user for anything the repository requires (the labels, the milestone, the form fields).
- Moving to the review state: a label, milestone or release note that must not be empty, if any (fill it even when the item is marked not customer-relevant).

## Agent tooling parent

In this repository: none. Agent tooling issues are ordinary issues.

## Reviewers

Candidates for chapter 15, by GitHub login (a team as `Panaith/{team}`). The owner of the tooling is `Panagiotis Sarridis`. Which reviews are required comes from the branch protection or ruleset on `master` and the `CODEOWNERS` file, not from the PR.

Merge method and squash message: by default GitHub's squash message is the commit title and message (one commit) or the PR title with the list of commits; the repository setting may use the PR title and description instead. The merge method is a project fact (see the marker below).

| Person | GitHub login |
|---|---|
| Panagiotis Sarridis | `Panaith` (checked 2026-10-23) |

In this repository (checked 2026-10-23): `master` has no branch protection, no ruleset and no `CODEOWNERS` file, so no review is required; `Panaith` is the only collaborator. Merge method: **squash**. Squash commit setting: title "commit or PR title" (one commit: its title; several: the PR title), message "commit messages".

## Review threads

Who resolves a review thread on a PR: by default the author resolves it once the fix is pushed, and the reviewer reopens it if the fix is not right.

In this repository: the default above; no ruleset requires resolved conversations.

## Work item fields

What the parts of an issue change in this workflow (chapter 4). Customer names and test data files are work data: `PERMISSIONS.md`, section 2.

| Field | What it changes |
|---|---|
| Developer Notes comment, the Progress comment, linked branches and PRs, release tags | Resumed or reopened work: what is already done and shipped (ch. 5, 12); the Developer Notes are merged, not overwritten (ch. 10), and the existing Progress comment is updated (ch. 11) |
| Assignees, author | Whom to ask when something is unclear |
| Version or environment answers of the issue form (a bug) | The version it was found in (ch. 7) |
| Labels for severity or priority (for example `priority: high`), milestone | Risk tolerance and test depth: size class (ch. 7), tests (ch. 8); the milestone names the release it belongs to |
| State, state reason, board Status | A closed or reopened issue, or a Status before the approved one: say so before planning |
| Release notes section or label | Text the product owner already wrote is the expected behaviour; ch. 15 extends it instead of replacing it |

Skip: board layout fields, iteration or sprint fields, effort estimates, bot comments and automation labels.

In this repository: only GitHub's default labels. `bug` and `enhancement` set the kind; `documentation` marks a change to documentation only; `duplicate`, `invalid`, `wontfix` and `question` mean the issue may not need code: say so before planning. Skip `good first issue` and `help wanted`.
