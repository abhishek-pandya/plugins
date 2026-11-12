---
name: reflect
description: Spawn three parallel review subagents over the active transcript, surface learnings, and route each to a concrete edit on an existing skill. Use when the user says reflect.
disable-model-invocation: true
---

# Reflect

Mine the current conversation for durable learnings, then route them into skill edits.

Invoke when the user says "reflect" or "/reflect". Skip when the conversation is trivial, off-topic, or already covered by an existing skill the parent followed correctly. One-offs are not learnings.

## 1. Locate the active transcript

Before fanning out, find this conversation's transcript in the `agent-transcripts/` directory the system prompt names. Do not glob across `~/.cursor/projects/*/`. That crosses workspace boundaries and reads private chats from unrelated projects.

```bash
ls -t <agent-transcripts>/*.jsonl <agent-transcripts>/*/*.jsonl <agent-transcripts>/*/subagents/*.jsonl 2>/dev/null | head -10
```

That covers the legacy flat (`<id>.jsonl`), current nested (`<id>/<id>.jsonl`), and subagent (`<parent>/subagents/<child>.jsonl`) layouts. For each candidate, read the first JSONL line. Take the file whose `message.content[0].text` contains the conversation's opening user prompt. If none matches, pass a tight digest of the session instead.

## 2. Spawn three reviewers in parallel

One message, three `Task` calls, `subagent_type: generalPurpose`, agent mode (`readonly: false`). Reviewers need MCP access to look up the tickets, chat threads, and observability traces the transcript references, and readonly strips MCPs.

Each reviewer and the synthesizer name a role line in the `pstack-models.mdc` rule and a default. Set `model` to that line's value, or to the default if the rule or the line is missing. Leave `model` unset when the value is `auto` or `inherit-parent`. If the Task tool rejects a slug, use the default and say so. If it rejects the default, use the closest valid slug of the same family from its error message. Pick each subagent's reasoning level and fast or standard speed for the job, per the Subagents section of the **poteto-mode** skill.

| Lens | Role line | Default `model` | Prompt template |
|---|---|---|---|
| Judgment | `reflect judgment, divergent, synthesizer` | `claude-opus-5-5-xhigh` | `references/judgment-reviewer.md` |
| Tooling | `reflect tooling` | `grok-4.7-xhigh-fast` | `references/tooling-reviewer.md` |
| Divergent | `reflect judgment, divergent, synthesizer` | `claude-opus-5-5-xhigh` | `references/divergent-reviewer.md` |

Pass each template verbatim, substituting the transcript path or digest where marked. Reviewers return findings in the `Task` response body.

## 3. Synthesize

One `Task` call, `subagent_type: generalPurpose`, `model` from the `reflect judgment, divergent, synthesizer` line (default `claude-opus-5-5-xhigh`), agent mode (`readonly: false`). It spot-verifies citations through MCP, and readonly strips MCPs. Pass `references/synthesizer.md` verbatim, with each reviewer's full output inlined where marked. It returns an Accepted / Rejected / Backlog list.

## 4. Structural enforcement check

Move any Accepted item that a lint rule, script, metadata flag, or runtime check would enforce more reliably to Backlog. See the **encode-lessons-in-structure** principle skill.

## 5. Apply

Present the synthesizer's full Accepted / Rejected / Backlog output and wait for explicit approval before applying any Accepted edit. The user picks the subset and may redirect routings. Skill changes affect every future agent in the org. Do not auto-apply.

File each Backlog item to your team's devex or backlog tracker without waiting. Only the Accepted list waits for approval.

Never edit a skill under `plugins/cache/`. Before approval, show the source destination. For pstack skills in this fork, use `pstack/skills/<skill>/` in the fork checkout and follow the publishing steps below. For unrelated skills, copy to `.cursor/skills/<skill>/` or `~/.cursor/skills/<skill>/` after approval. Do not publish unrelated skills through this workflow.

If no skill-authoring skill is available, follow `../poteto-mode/playbooks/authoring-a-skill.md`.

Follow each approved row's Routing exactly:

- Trivial existing-skill edit (a one-line bullet, a tightened sentence, a stale fact corrected): the parent does it directly.
- Substantive existing-skill edit (a new section, a new pattern table, more than ~10 lines): hand to an available skill-authoring skill and run its draft / test / iterate loop.
- `tune description: <skill path>` (the skill exists but didn't trigger when it should have): hand to the available skill-authoring skill and run its description-optimization loop.
- `new skill: <kebab-name>`: hand creation to the available skill-authoring skill. Do not invent the shape ad hoc.

If your environment ships a SKILL.md validator, run it on every touched skill before declaring done.

### Publish approved pstack edits

Approved pstack edits in this fork include publishing the fork and its dotfiles pin, unless the user requests local-only changes. Existing explicit approval counts. Do not ask again for the same edits.

1. Reuse a checkout of `https://github.com/abhishek-pandya/plugins.git`, or clone it into a writable workspace. A sparse checkout needs `pstack`, `harness/pstack`, and `scripts`. Verify the remote before editing. Fetch and start from the fork's current `main`. Preserve unrelated work with a separate worktree when needed.
2. Edit `pstack/skills/<skill>/`, never the generated `harness/pstack/claude-code/` files directly. For each release, increment the fork prerelease suffix, such as `0.15.36-abhi.1` to `0.15.36-abhi.2`, in both `pstack/plugin.json` and `pstack/.cursor-plugin/plugin.json`. If there is no suffix, append `-abhi.1`. Run `node scripts/generate-pstack-harness-trees.mjs`, then its `--check` mode and checks appropriate to the skill change.
3. Commit only the intended source, manifest, and generated changes. Push to the fork's `main` without force. Verify the published commit exists on the remote before updating dotfiles. If a concurrent push wins, integrate it and rerun the checks before retrying.
4. Reuse or clone `https://github.com/abhishek-pandya/dotfiles.git`. Start from its current `main`, using a separate worktree when needed to preserve unrelated work. In `.claude-plugin/marketplace.json`, find the `pstack` entry and verify its source points to the fork and `./harness/pstack/claude-code`. Set `source.sha` to the published fork commit. Update the documented pin and plugin version in `ai/README.md`. Run `python3 tests/test-install-skills.py`, `python3 tests/test-bootstrap.py`, and `git diff --check`. Commit those dotfiles changes and push without force.
5. Verify both remote commits and the pin relationship. Report both commit links. If either publication fails, report the completed step and the remaining step. Never report delivery as complete after only the fork push.

New Ona instances receive the published version when normal bootstrap uses the latest dotfiles checkout. Existing enabled installations are skipped by `install-skills.sh`; a bootstrap rerun alone does not upgrade them. Use the plugin refresh instructions in dotfiles' `ai/README.md` when an existing installation must update, then start a new agent session. Do not add a recurring updater or bypass an organization's plugin policy.

## 6. Summarize for the user

Short list, no preamble:

- Edits applied: `<skill path>`. What changed, one line each.
- New skills created: `<skill path>`. One line each (rare).
- Backlog filed to the devex tracker: `<issue title>` (`<tags>`). One line each.
- Dropped: one line per rejected finding + reason from the synthesizer.
