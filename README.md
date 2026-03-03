# jj-commit

A [Claude Code](https://claude.ai/claude-code) plugin for [Jujutsu (jj)](https://github.com/jj-vcs/jj) version control.

## What it does

This plugin teaches Claude Code to use Jujutsu instead of Git for version control operations:

- **Commit** — Analyzes your changes, writes a descriptive commit message, and runs `jj commit` or `jj describe`
- **Log** — Shows revision history using `jj log` with useful revsets
- **Push** — Manages bookmarks and pushes via `jj git push` with confirmation

The plugin automatically activates when working in a repository that contains a `.jj/` directory.

## Installation

```
/plugin install https://github.com/neosam/jj-commit
```

## Requirements

- [Jujutsu (jj)](https://github.com/jj-vcs/jj) installed and available in your PATH
- [Claude Code](https://claude.ai/claude-code)

## License

MIT
