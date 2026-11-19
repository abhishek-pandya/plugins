---
name: implement-jira-ticket
description: Use this skill when implementing a Jira ticket that is well-scoped. Not every single decision needs to be made, but the goal and how it gets implemented and how it relates to other changes should be clear. If not, do not use this skill and instead mention to the user that the ticket does not have enough information to use this flow.
---

# Implement Jira ticket

Take a well-scoped Jira ticket to a complete draft PR with all review comments addressed and all CI checks passing. Keep the PR in draft. Do not merge or mark it ready.

## 1. Investigate and confirm

Follow [abhi-mode](../abhi-mode/SKILL.md). Use the Jira CLI `acli` to fetch ticket information, including the ticket details, comments, and related tickets. If `acli` is unavailable or authentication fails, report the blocker before using another access method. Read the full ticket, comments, engineering spec, linked context, and related tickets. Inspect the relevant code and related changes to establish dependencies and scope.

The goal, implementation approach, and relationship to other changes must be clear; minor implementation decisions may remain open. If required context is missing or these points remain unclear, tell the user the ticket does not have enough information to use this flow and identify the gaps. Do not proceed with implementation under this skill.

Identify the branches and PRs for prerequisite tickets and check whether they have merged. If this ticket depends on unmerged work, propose basing its branch on the dependency branch. If dependencies already form a stack, use the branch that contains all required changes. If more than one prerequisite branch remains unmerged, warn the user during investigation: name the branches and PRs, explain the risk of repeated rebases, conflicts, and review churn, and propose a base and sequencing plan. If no single branch contains all prerequisites, make that explicit rather than silently combining branches.

Present your understanding of the changes needed to accomplish the ticket, including scope, dependencies, proposed base branch, any dependency warning, and how completion will be verified. Wait for the user's confirmation before proceeding.

## 2. Implement

Use [poteto-mode](../poteto-mode/SKILL.md) and its matching playbook to implement and verify the confirmed scope. Continue this flow when abhi-mode routes back here; do not restart investigation. Defer the playbook's Opening a PR step until step 5 below. If a draft PR already exists, use it.

Base the work on the confirmed dependency branch while its prerequisite changes remain unmerged; use the repository's normal base once those changes have merged. Target the draft PR at the dependency branch when stacking, and review the ticket's diff against that base. Follow the confirmed sequencing plan when multiple prerequisite branches are involved.

## 3. Independent review

Once implementation and verification satisfy you, launch a fresh reviewer in a different agent harness. From Codex, launch Claude Code; from Claude Code, launch Codex. A same-harness subagent does not satisfy this requirement. If the other harness is unavailable, report that blocker and ask the user how to proceed rather than silently substituting yourself.

Give the reviewer access to the ticket and the branch diff or existing PR. Its only task prompt is the following, with the placeholders replaced by the ticket, review target, and absolute skill path. Do not pass your implementation rationale, conclusions, or suggested findings.

> Investigate Jira ticket <ticket>. Evaluate whether <branch diff or PR> accomplishes the ticket's goals, and perform a thorough PR review using the interrogate skill at <absolute path to ../interrogate/SKILL.md>.

## 4. Address findings

Evaluate the review findings against the ticket and code. Fix worthwhile findings and verify the affected behavior. Record concrete reasons for dismissing findings that do not warrant a change. Resolve remaining actionable findings before proceeding.

## 5. Open the draft PR and babysit

Use [Opening a PR](../poteto-mode/playbooks/opening-a-pr.md) to open or update the draft PR. Then run [Babysit](../poteto-mode/playbooks/babysit.md) in `drive` mode. Invoking this skill requests that babysit pass, including addressing and replying to Greptile and other review comments; no separate babysit request is needed. Use `drive` even for a small or docs-only ticket.

Prefer Claude Code for babysitting when available, to use its monitoring capabilities. Give the babysitter the PR, confirmed scope, and this skill's completion condition. Keep a single babysitter responsible for the PR; if Claude Code is unavailable, babysit in the current harness and report the substitution.

Check each review claim against the code. Fix valid issues, explain dismissals, and resolve addressed threads where supported. After each push, wait for CI and configured reviews on the latest revision, then address any new findings. Do not claim completion while a review or check is pending, failing, or blocked.

This flow's stopping condition overrides Babysit's merge-ready condition: the PR remains a draft, all CI checks pass on the latest revision, configured reviews have completed, and no review comments remain unaddressed. Historical comments may remain visible. Do not mark the PR ready to obtain a watcher `READY` verdict. Use status inspection if the watcher cannot express this draft completion condition.

If an external blocker prevents completion, report the blocker and what remains instead of claiming success. Finish with the draft PR link, implemented scope, verification results, and final review and CI status.
