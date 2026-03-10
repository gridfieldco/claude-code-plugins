# Gridfield Claude Code Plugins

A plugin marketplace for Claude Code with developer productivity tools.

## Installation

```shell
/plugin marketplace add gridfieldco/claude-code-plugins
```

## Available Plugins

### commit-prep
Review staged git changes and draft a conventional commit message.

```shell
/plugin install commit-prep@gridfield-tools
/commit-prep
```

### explain-error
Paste an error message and get a concise explanation with fix suggestions.

```shell
/plugin install explain-error@gridfield-tools
/explain-error <paste your error>
```

## Adding Plugins

Want to contribute? Add your plugin under `plugins/` and register it in `.claude-plugin/marketplace.json`.
