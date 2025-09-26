<div align="center">
    <img src="./media/logo_small.webp"/>
    <h1>🌱 Spec Kit Ext</h1>
    <h3><em>Extends GitHub's Spec Kit to thrive in complex and brownfield environments.</em></h3>
</div>

<p align="center">
    <strong>An effort to allow organizations to focus on product scenarios rather than writing undifferentiated code with the help of Spec-Driven Development.</strong>
</p>

[![Release](https://github.com/danwashusen/spec-kit-ext/actions/workflows/release.yml/badge.svg)](https://github.com/github/spec-kit/actions/workflows/release.yml)

---

## Table of Contents

- [🤔 What is Spec-Driven Development?](#-what-is-spec-driven-development)
- [⚡ Get started](#-get-started)
- [🤖 Supported AI Agents](#-supported-ai-agents)
- [🔧 Specify Ext CLI Reference](#-specify-ext-cli-reference)
  - [Commands](#commands)
  - [`specify-ext init` Arguments & Options](#specify-ext-init-arguments--options)
  - [Examples](#examples)
  - [Available Slash Commands](#available-slash-commands)
  - [Environment Variables](#environment-variables)
- [🧩 Configuration](#-configuration)
- [📚 Core philosophy](#-core-philosophy)
- [🔧 Prerequisites](#-prerequisites)
- [📖 Learn more](#-learn-more)
- [📋 Detailed process](#-detailed-process)
- [🔍 Troubleshooting](#-troubleshooting)
  - [Git Credential Manager on Linux](#git-credential-manager-on-linux)
- [👥 Maintainers](#-maintainers)
- [💬 Support](#-support)
- [🙏 Acknowledgements](#-acknowledgements)
- [📄 License](#-license)

## 🤔 What is Spec-Driven Development?

Spec-Driven Development **flips the script** on traditional software development. Specifications become living assets that drive implementation, making the intent of the system just as important as the code itself.

This fork extends the core Spec Kit philosophy to thrive in complex and brownfield environments. It layers in:

- Richer context loading for the Spec Kit command suite (`/specify`, `/plan`, `/tasks`, `/implement`, etc.) so you can feed in the documents that already describe your system — PRDs, architecture decision records, threat models, runbooks, and more.
- Project scaffolding tuned for teams with existing estates, where incremental modernization and integration work is the norm.
- Opinionated defaults that encourage documenting why things exist today before you change them, keeping legacy constraints visible while you plan the future.

Use this extension when you want the structure of Spec Kit but need the flexibility to reason about deeply interconnected systems, historical context, and the real-world artifacts that come with mature products.

## ⚡ Get started

Start with the original Spec Kit in action video: [video overview](https://www.youtube.com/watch?v=a9eR1xsfvHg&pp=0gcJCckJAYcqIYzv)!

Head over to the [Quick Start Guide](docs/quickstart.md) for installation steps, slash-command usage, and a full walkthrough of the Spec Kit workflow.

For additional background, the [comprehensive methodology](./spec-driven.md) explains the philosophy behind each phase.

## 🤖 Supported AI Agents

| Agent                                                     | Support | Notes                                             |
|-----------------------------------------------------------|---------|---------------------------------------------------|
| [Claude Code](https://www.anthropic.com/claude-code)      | ✅ |                                                   |
| [GitHub Copilot](https://code.visualstudio.com/)          | ✅ |                                                   |
| [Gemini CLI](https://github.com/google-gemini/gemini-cli) | ✅ |                                                   |
| [Cursor](https://cursor.sh/)                              | ✅ |                                                   |
| [Qwen Code](https://github.com/QwenLM/qwen-code)          | ✅ |                                                   |
| [opencode](https://opencode.ai/)                          | ✅ |                                                   |
| [Windsurf](https://windsurf.com/)                         | ✅ |                                                   |
| [Kilo Code](https://github.com/Kilo-Org/kilocode)         | ✅ |                                                   |
| [Auggie CLI](https://docs.augmentcode.com/cli/overview)   | ✅ |                                                   |
| [Roo Code](https://roocode.com/)                          | ✅ |                                                   |
| [Codex CLI](https://github.com/openai/codex)              | ⚠️ | Codex [does not support](https://github.com/openai/codex/issues/2890) custom arguments for slash commands.  |

## 🔧 Specify Ext CLI Reference

The `specify-ext` command supports the following options:

### Commands

| Command     | Description                                                    |
|-------------|----------------------------------------------------------------|
| `init`      | Initialize a new Specify project from the latest template      |
| `check`     | Check for installed tools (`git`, `claude`, `gemini`, `code`/`code-insiders`, `cursor-agent`, `windsurf`, `qwen`, `opencode`, `codex`) |

### `specify-ext init` Arguments & Options

| Argument/Option        | Type     | Description                                                                  |
|------------------------|----------|------------------------------------------------------------------------------|
| `<project-name>`       | Argument | Name for your new project directory (optional if using `--here`, or use `.` for current directory) |
| `--ai`                 | Option   | AI assistant to use: `claude`, `gemini`, `copilot`, `cursor`, `qwen`, `opencode`, `codex`, `windsurf`, `kilocode`, `auggie`, or `roo` |
| `--script`             | Option   | Script variant to use: `sh` (bash/zsh) or `ps` (PowerShell)                 |
| `--ignore-agent-tools` | Flag     | Skip checks for AI agent tools like Claude Code                             |
| `--no-git`             | Flag     | Skip git repository initialization                                          |
| `--here`               | Flag     | Initialize project in the current directory instead of creating a new one   |
| `--force`              | Flag     | Force merge/overwrite when initializing in current directory (skip confirmation) |
| `--skip-tls`           | Flag     | Skip SSL/TLS verification (not recommended)                                 |
| `--debug`              | Flag     | Enable detailed debug output for troubleshooting                            |
| `--github-token`       | Option   | GitHub token for API requests (or set GH_TOKEN/GITHUB_TOKEN env variable)  |

### Examples

```bash
# Basic project initialization
specify-ext init my-project

# Initialize with specific AI assistant
specify-ext init my-project --ai claude

# Initialize with Cursor support
specify-ext init my-project --ai cursor

# Initialize with Windsurf support
specify-ext init my-project --ai windsurf

# Initialize with PowerShell scripts (Windows/cross-platform)
specify-ext init my-project --ai copilot --script ps

# Initialize in current directory
specify-ext init . --ai copilot
# or use the --here flag
specify-ext init --here --ai copilot

# Force merge into current (non-empty) directory without confirmation
specify-ext init . --force --ai copilot
# or 
specify-ext init --here --force --ai copilot

# Skip git initialization
specify-ext init my-project --ai gemini --no-git

# Enable debug output for troubleshooting
specify-ext init my-project --ai claude --debug

# Use GitHub token for API requests (helpful for corporate environments)
specify-ext init my-project --ai claude --github-token ghp_your_token_here

# Check system requirements
specify-ext check
```

### Available Slash Commands

After running `specify-ext init`, your AI coding agent will have access to these slash commands for structured development:

| Command         | Description                                                           |
|-----------------|-----------------------------------------------------------------------|
| `/constitution` | Create or update project governing principles and development guidelines |
| `/specify`      | Define what you want to build (requirements and user stories)        |
| `/clarify`      | Clarify underspecified areas (must be run before `/plan` unless explicitly skipped; formerly `/quizme`) |
| `/plan`         | Create technical implementation plans with your chosen tech stack     |
| `/tasks`        | Generate actionable task lists for implementation                     |
| `/analyze`      | Cross-artifact consistency & coverage analysis (run after /tasks, before /implement) |
| `/implement`    | Execute all tasks to build the feature according to the plan         |
| `/audit`        | Governance-focused review that enforces quality gates and policy controls before release |

### Environment Variables

| Variable         | Description                                                                                    |
|------------------|------------------------------------------------------------------------------------------------|
| `SPECIFY_FEATURE` | Override feature detection for non-Git repositories. Set to the feature directory name (e.g., `001-photo-albums`) to work on a specific feature when not using Git branches.<br/>**Must be set in the context of the agent you're working with prior to using `/plan` or follow-up commands. |

## 🧩 Configuration

Refer to the [configuration guide](docs/configuration.md) for lookup order, schema details, and how each command consumes `SPEC_KIT_CONFIG`.

## 📚 Core philosophy

Spec-Driven Development is a structured process that emphasizes:

- **Intent-driven development** where specifications define the "_what_" before the "_how_"
- **Rich specification creation** using guardrails and organizational principles
- **Multi-step refinement** rather than one-shot code generation from prompts
- **Heavy reliance** on advanced AI model capabilities for specification interpretation

## 🔧 Prerequisites

- **Linux/macOS** (or WSL2 on Windows)
- AI coding agent: [Claude Code](https://www.anthropic.com/claude-code), [GitHub Copilot](https://code.visualstudio.com/), [Gemini CLI](https://github.com/google-gemini/gemini-cli), [Cursor](https://cursor.sh/), [Qwen CLI](https://github.com/QwenLM/qwen-code), [opencode](https://opencode.ai/), [Codex CLI](https://github.com/openai/codex), or [Windsurf](https://windsurf.com/)
- [uv](https://docs.astral.sh/uv/) for package management
- [Python 3.11+](https://www.python.org/downloads/)
- [Git](https://git-scm.com/downloads)

If you encounter issues with an agent, please open an issue so we can refine the integration.

## 📖 Learn more

- **[Complete Spec-Driven Development Methodology](./spec-driven.md)** - Deep dive into the full process
- **[Detailed Walkthrough](#-detailed-process)** - Step-by-step implementation guide
- **[Local Development Guide](docs/local-development.md)** - Run the CLI from source, editable installs, uvx flows, and using locally built template ZIPs

---

## 📋 Detailed process

See the [Quick Start Guide](docs/quickstart.md) for the full step-by-step walkthrough, including CLI setup, prompt examples, and screenshots.

---

## 🔍 Troubleshooting

### Git Credential Manager on Linux

If you're having issues with Git authentication on Linux, you can install Git Credential Manager:

```bash
#!/usr/bin/env bash
set -e
echo "Downloading Git Credential Manager v2.6.1..."
wget https://github.com/git-ecosystem/git-credential-manager/releases/download/v2.6.1/gcm-linux_amd64.2.6.1.deb
echo "Installing Git Credential Manager..."
sudo dpkg -i gcm-linux_amd64.2.6.1.deb
echo "Configuring Git to use GCM..."
git config --global credential.helper manager
echo "Cleaning up..."
rm gcm-linux_amd64.2.6.1.deb
```

## 👥 Maintainers

- Dan Washusen ([@danwashusen](https://github.com/danwashusen))

## 💬 Support

For support, please open a [GitHub issue](https://github.com/danwashusen/spec-kit-ext/issues/new). We welcome bug reports, feature requests, and questions about using Spec-Driven Development.

## 🙏 Acknowledgements

This project is heavily influenced by and based on the work and research of [John Lam](https://github.com/jflam) and is mostly authored by the Spec Kit team:
- Den Delimarsky ([@localden](https://github.com/localden))
- John Lam ([@jflam](https://github.com/jflam))

## 📄 License

This project is licensed under the terms of the MIT open source license. Please refer to the [LICENSE](./LICENSE) file for the full terms.
