# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Ansible playbook for automating macOS development environment setup. It's a fork of geerlingguy/mac-dev-playbook with personal customizations.

## Commands

```bash
# Run full playbook
ansible-playbook main.yml --ask-become-pass

# Run specific tags only
ansible-playbook main.yml -K --tags "homebrew,dotfiles"

# Syntax check
ansible-playbook main.yml --syntax-check

# Dry run (check mode)
ansible-playbook main.yml --ask-become-pass --check

# Install role/collection dependencies
ansible-galaxy install -r requirements.yml

# Lint
yamllint . && ansible-lint
```

**Available tags:** `dotfiles`, `homebrew`, `mas`, `osx`, `extra-packages`, `dock`, `sublime-text`, `terminal`, `sudoers`, `post`

## Architecture

### Configuration Layering
- `default.config.yml` - Upstream defaults (committed to git)
- `config.yml` - Personal overrides (git-ignored, never committed)

The playbook loads defaults first, then overlays `config.yml` via pre-tasks. This pattern enables clean upstream syncs without merge conflicts.

**Note:** In this checkout, `config.yml` is a symlink to iCloud Drive
(`~/Library/Mobile Documents/com~apple~CloudDocs/Developer/config files/config.yml`)
so the personal config is shared across machines. On a fresh clone the symlink
exists but may dangle until iCloud syncs — do not replace it with a regular file.

### Playbook Flow (main.yml)
1. **Pre-tasks**: Load configuration files
2. **Roles** (external):
   - `elliotweiser.osx-command-line-tools` - Xcode CLI tools
   - `geerlingguy.mac.homebrew` - Homebrew packages/casks
   - `geerlingguy.dotfiles` - Dotfiles management
   - `geerlingguy.mac.mas` - Mac App Store apps
   - `geerlingguy.mac.dock` - Dock configuration
3. **Tasks** (inline in `tasks/`):
   - sudoers, terminal, osx, extra-packages, sublime-text
4. **Post-provision**: Custom tasks via glob pattern

### Key Directories
- `roles/` - External roles installed via ansible-galaxy
- `tasks/` - Custom task files for macOS-specific operations
- `files/` - Static files (Terminal themes, Sublime settings)
- `templates/` - Jinja2 templates
- `tests/` - CI test configuration

### Conditional Execution
Most features are controlled by boolean flags in config (e.g., `configure_dotfiles`, `configure_osx`, `configure_dock`). Roles and tasks use `when:` conditions based on these flags.

## Dependencies

External roles/collections defined in `requirements.yml`:
- `elliotweiser.osx-command-line-tools`
- `geerlingguy.dotfiles`
- `geerlingguy.mac` collection (homebrew, mas, dock roles)

## CI/CD

GitHub Actions runs on macOS 14 and 15:
1. Lint job (yamllint + ansible-lint)
2. Integration tests with idempotence check (second run must have 0 changes)

## Upstream Sync

See `UPSTREAM-SYNC.md` for syncing with geerlingguy/mac-dev-playbook. Personal config in `config.yml` stays local; only `default.config.yml` comes from upstream.
