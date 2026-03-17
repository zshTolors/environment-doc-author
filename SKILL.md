---
name: environment-doc-author
description: Create or update environment baseline documents from real local machine state. Use when Codex, another AI agent, a CLI runner, or OpenClaw needs to inspect installed tools, env vars, services, runtimes, compilers, startup scripts, or build/run/test/debug/migration tasks, then write or refresh `ENVIRONMENT_POLICY.md`, `AGENTS.md` environment guardrails, related skill instructions, or a machine-readable baseline JSON.
---

# Environment Doc Author

## Overview

Use this skill to turn the current machine state into reusable environment guardrails.
Probe the machine first, keep a machine-readable baseline JSON, and render human-readable policy fragments from verified facts only.

This skill supports two standalone implementations:

- JavaScript
  - `scripts/detect_environment.js`
  - `scripts/render_environment_docs.js`
  - Pure Node.js. No Python dependency.
- Python
  - `scripts/detect_environment.py`
  - `scripts/render_environment_docs.py`
  - Requires Python 3.7+.

## Workflow

### 1. Read the existing baseline before probing

- If a machine-level policy already exists, read it first and treat it as the current baseline.
- Prefer a machine-readable JSON baseline when available. Recommended names:
  - `environment-baseline.json`
  - `ENVIRONMENT_POLICY.md`
  - `AGENTS.md` or a dedicated environment snippet
- If your team or machine already has a master environment policy, treat it as the primary source of truth before probing anything else.

### 2. Create the initial baseline when none exists

- Run one implementation against the current machine and save the result as JSON.
- Render the JSON into the human-readable docs that the repository or agent stack needs.
- Keep the JSON and human docs in sync. Do not maintain only one of them.
- Both implementations auto-detect the current machine language and generate Chinese or English docs. If the language cannot be recognized, they default to English.

JavaScript:

```bash
node scripts/detect_environment.js --output ./environment-baseline.json
node scripts/render_environment_docs.js ./environment-baseline.json \
  --policy-out ./ENVIRONMENT_POLICY.md \
  --agents-out ./AGENTS.environment.md \
  --skill-snippet-out ./SKILL.environment.md
```

Python:

```bash
python3 scripts/detect_environment.py --output ./environment-baseline.json
python3 scripts/render_environment_docs.py ./environment-baseline.json \
  --policy-out ./ENVIRONMENT_POLICY.md \
  --agents-out ./AGENTS.environment.md \
  --skill-snippet-out ./SKILL.environment.md
```

Windows note for Python:

- Do not assume the default `python` command is new enough.
- Verify that the selected interpreter is Python 3.7+ before running the Python entry points.

### 3. Use targeted verification when a baseline already exists

- Do not re-probe everything by default when only one tool or task changed.
- If a task mentions a tool not covered by the current baseline, probe that tool first, then update the docs.
- If the current machine state does not match the baseline, record the verified change and refresh both JSON and human docs.

JavaScript:

```bash
node scripts/detect_environment.js \
  --baseline ./environment-baseline.json \
  --tools java,maven \
  --output ./environment-baseline.json

node scripts/render_environment_docs.js ./environment-baseline.json \
  --policy-out ./ENVIRONMENT_POLICY.md \
  --agents-out ./AGENTS.environment.md
```

Python:

```bash
python3 scripts/detect_environment.py \
  --baseline ./environment-baseline.json \
  --tools java,maven \
  --output ./environment-baseline.json

python3 scripts/render_environment_docs.py ./environment-baseline.json \
  --policy-out ./ENVIRONMENT_POLICY.md \
  --agents-out ./AGENTS.environment.md
```

### 4. Extend the probe set only when the task requires it

- The built-in detector covers common local tools and core env vars:
  - Git
  - Java
  - `javac`
  - Maven
  - Gradle
  - Node
  - npm
  - pnpm
  - Yarn
  - Python
  - Go
  - Rust (`rustc`, Cargo, Rustup)
  - Clang, Clang++, GCC, G++, CMake, Make, Ninja
  - Ruby, RubyGems, Bundler
  - PHP, Composer
  - .NET SDK
  - Docker
  - kubectl
  - `JAVA_HOME`
  - `MAVEN_HOME`
  - `GRADLE_HOME`, `GOROOT`, `GOPATH`, `CARGO_HOME`, `RUSTUP_HOME`, `DOTNET_ROOT`, `CC`, `CXX`
  - common runtime managers such as `NVM_DIR`, `PYENV_ROOT`, `RBENV_ROOT`, `SDKMAN_DIR`, `HOMEBREW_PREFIX`, `VIRTUAL_ENV`
  - important `PATH` entries
- For services, databases, startup scripts, or project-specific tooling, add an extra probe file and run a targeted scan.
- Read [probe-file.md](./references/probe-file.md) only when you need extra probes.

### 5. Apply the hard rules consistently

- Do not invent tools, versions, install roots, services, or env vars.
- Verify executables with absolute paths and version commands whenever possible.
- Distinguish shims, wrappers, symlinks, aliases, and launchers from real install directories.
- If a tool is missing:
  1. Tell the user which tool is missing.
  2. Explain why it is needed.
  3. Ask for the executable path or install directory after installation.
  4. Prefer user-managed installation.
  5. Only install it yourself if the user explicitly requests and authorizes that action.
  6. Update both the JSON baseline and the human docs after verification.
- If a command would modify the machine environment, stop and obtain explicit approval first.

## Output Contract

- Keep these roles distinct:
  - `environment-baseline.json`: machine-readable source of truth for detected facts
  - `ENVIRONMENT_POLICY.md`: human-readable policy and approved environment inventory
  - `AGENTS.environment.md` or `AGENTS.md` snippet: hard rules for agents
  - `SKILL.environment.md` snippet: environment section that can be reused in other skills
- Read [document-contracts.md](./references/document-contracts.md) when you need the exact document shape, language behavior, or update checklist.

## Resources

### `scripts/detect_environment.js`

Standalone Node.js detector. Probes the current machine, merges with an existing baseline when requested, and emits normalized JSON.

### `scripts/render_environment_docs.js`

Standalone Node.js renderer. Generates localized `ENVIRONMENT_POLICY.md`, `AGENTS` snippets, and skill snippets from the baseline JSON.

### `scripts/detect_environment.py`

Python detector. Probes the current machine, merges with an existing baseline when requested, and emits normalized JSON.

### `scripts/render_environment_docs.py`

Python renderer. Generates localized `ENVIRONMENT_POLICY.md`, `AGENTS` snippets, and skill snippets from the baseline JSON.

### `references/document-contracts.md`

Use when you need the exact structure and maintenance rules for the generated docs.

### `references/probe-file.md`

Use when you need to extend the default detector for services, databases, compilers, startup scripts, or project-specific tools.
