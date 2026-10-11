# Vanami plugins for Claude Code

A [Claude Code plugin marketplace](https://code.claude.com/docs/en/plugins) with Vanami's plugins.

Add the marketplace once:

```sh
claude plugin marketplace add vanami/claude-plugins
```

Then install the plugins you want:

| Plugin | Install | What it does |
| --- | --- | --- |
| [isms](plugins/isms) | `claude plugin install isms@vanami` | Answers the ISO 27001 auditor's questions in your ISMS at [isms.vanami.com](https://isms.vanami.com) from your code and infrastructure |

Update with `claude plugin marketplace update vanami`.

## Adding a plugin

1. Put it in `plugins/<name>/` with `.claude-plugin/plugin.json` (its `name` is `<name>`), and skills, agents and `.mcp.json` at the plugin's root.
2. Add an entry to `.claude-plugin/marketplace.json` with `"source": "./plugins/<name>"`.
3. Check with `claude plugin validate .` and `claude plugin validate plugins/<name>`, and add a row to the table above.
