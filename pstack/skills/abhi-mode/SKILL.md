---
name: abhi-mode
description: "Use for /abhi-mode, Abhi mode, or requests to work in Abhi's style. Apply his coding, planning, review, and explanation preferences."
disable-model-invocation: true
---

# Abhi mode

Apply these preferences to the current task. Use repository guidance for local conventions.

## Understand and plan

- Respect the requested phase. A request for an explanation, investigation, or plan does not authorize implementation.
- For substantial changes, establish scope and discuss high-level designs before detailing the implementation plan. Identify consequential assumptions and unresolved product decisions.
- Map the relevant responsibilities, callers, lifecycle cases, and dependencies. For frontend changes, show component ownership and query ordering. Check existing behavior and analogous implementations before proposing changes.
- Make plans concrete. Show before and after snippets of the same code against the intended base branch. Use actual types and function signatures. Include input/output examples where they clarify behavior. Keep prose focused on decisions and why each change is needed.
- Report deviations from the agreed plan, changed assumptions, and unforeseen complications when presenting the implementation.

## Write readable code

- Keep functions focused and put early exits near the beginning. Move responsibilities into their owning services instead of extending an already overloaded class.
- Extract meaningful helpers and named predicates when they make logic easier to follow. Compute repeated expressions once. Simplify nested conditionals rather than compressing them into convoluted expressions.
- Name concepts by their domain meaning. Distinguish rollout switches from setting values and existence from permission to view. Use explicit states when a sentinel would obscure meaning. Allow missing data when callers can handle it.
- Inspect existing services, utilities, domain types, and repository conventions before adding new machinery. Keep public APIs narrow and respect dependency direction.
- Justify interfaces, injection, wrappers, and shared components with a concrete need. Reuse useful abstractions; do not add layers that only forward arguments or fragment a simple type.
- Keep comments that explain nonobvious reasons for a choice, and update them when behavior changes.

## Review and verify

- Organize reviews by end-to-end flows through the changed code. Group relevant tests with each flow. Split substantial work into coherent commits or PRs, keeping unrelated cleanup separate.
- Explain what each proposed test proves. Favor meaningful product behavior and regression coverage. Remove overlap and avoid elaborate tests for trivial changes or temporary implementation details.
- Choose checks appropriate to the affected code and the user's requested verification scope. Verify the relevant real workflow where practical. Respect verification the user takes on, and state what remains unverified.
- Ground conclusions in decisive code, logs, or useful before/after measurements. Distinguish a hypothesis from demonstrated behavior. Explain what measurements include and reconcile conflicting counts.

## Prepare PRs

- Work in the user's current checkout. Never create or use a separate Git worktree unless the user explicitly asks, even if another skill or workflow recommends one.
- Open PRs as drafts. Mark them ready only when the user asks.
- Follow the repository's PR template and complete its sections. Keep motivation concise and connect it to the originating ticket or problem. Include concrete testing evidence and update the description when scope changes.
- Keep scratch planning notes out of implementation commits unless the user asks to include them. Honor explicit review, commit, and publication boundaries.

## Explain the work

- Start with purpose and product behavior, then explain the mechanism and logical relationships.
- Use concrete examples or the decisive code instead of broad architectural narration. Omit IDs and field lists when they do not help explain the concept.
- Use tables for comparisons and scope inventories, diagrams for relationships or execution order, and simple lists for a few links. Keep explanations concise without dropping the reasoning needed to assess the change.
