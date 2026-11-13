# Personal pstack plugins

This fork contains [pstack](pstack/README.md), including [poteto-mode](pstack/skills/poteto-mode/SKILL.md), [abhi-mode](pstack/skills/abhi-mode/SKILL.md), and their supporting workflows and principles.

See the [pstack README](pstack/README.md) for installation and usage.

## Repository structure

- `pstack/`: source skills, agents, documentation, and plugin manifests.
- `harness/pstack/claude-code/`: generated Claude Code distribution.
- `.cursor-plugin/marketplace.json`: Cursor marketplace containing pstack.
- `.claude-plugin/marketplace.json`: Claude Code marketplace containing pstack.
- `scripts/` and `schemas/`: generation and validation tools.

After changing pstack, regenerate the Claude Code distribution:

```sh
node scripts/generate-pstack-harness-trees.mjs
node scripts/generate-pstack-harness-trees.mjs --check
node scripts/check-pstack-docs.mjs
```

## License

See [pstack/LICENSE](pstack/LICENSE).
