# Claude Code Plugins

A collection of plugins for [Claude Code](https://claude.com/claude-code).

## Available Plugins

| Plugin | Description |
|--------|-------------|
| [debug-workflow](./plugins/debug-workflow/) | Systematic debugging workflow with hypothesis-driven investigation |

## Installation

Add plugins to your Claude Code configuration by specifying the path to the plugin directory:

```json
{
  "plugins": [
    "/path/to/cc-plugins/plugins/debug-workflow"
  ]
}
```

## Plugin Structure

Each plugin follows the Claude Code plugin structure:

```
plugins/<plugin-name>/
├── .claude-plugin/
│   └── plugin.json      # Plugin metadata
├── agents/              # Specialized agent definitions
│   └── *.md
├── commands/            # Command definitions
│   └── *.md
└── README.md            # Plugin documentation
```

## Contributing

To add a new plugin:

1. Create a new directory under `plugins/`
2. Add the required `.claude-plugin/plugin.json` file
3. Define agents in `agents/` and commands in `commands/`
4. Document usage in a `README.md`
5. Update this file to list the new plugin

## License

MIT
