# Environment Doc Author

[Chinese (Simplified)](./README.zh-CN.md)

Generate and maintain environment baseline documents from verified local machine state.
This repository ships two standalone implementations that detect the current environment, keep a machine-readable baseline, and render guardrail documents such as `ENVIRONMENT_POLICY.md`, `AGENTS` snippets, and reusable skill snippets.

## Implementations

- `scripts/detect_environment.js` / `scripts/render_environment_docs.js`
  - Pure Node.js implementation.
  - No Python dependency.
- `scripts/detect_environment.py` / `scripts/render_environment_docs.py`
  - Python implementation.
  - Requires Python 3.7+.

## Outputs

- `environment-baseline.json`
  - Machine-readable source of truth for detected tools, versions, env vars, PATH highlights, and targeted checks.
- `ENVIRONMENT_POLICY.md`
  - Human-readable policy for the current machine.
- `AGENTS.environment.md`
  - Short operational rules for agents.
- `SKILL.environment.md`
  - Reusable skill snippet derived from the same baseline.

## Requirements

- JavaScript entry points
  - Node.js 18+ recommended.
  - Python is not required.
- Python entry points
  - Python 3.7+ required.
  - On Windows, confirm which interpreter you are actually using before running the Python scripts.
  - If multiple Python versions are installed, prefer an explicit Python 3.7+ interpreter instead of assuming `python` points to the right one.

## Repository Layout

```text
.
|-- SKILL.md
|-- README.md
|-- README.zh-CN.md
|-- references/
|   |-- document-contracts.md
|   `-- probe-file.md
`-- scripts/
    |-- detect_environment.js
    |-- detect_environment.py
    |-- render_environment_docs.js
    `-- render_environment_docs.py
```

## Quick Start

### JavaScript

```bash
node scripts/detect_environment.js --output ./environment-baseline.json
node scripts/render_environment_docs.js ./environment-baseline.json \
  --policy-out ./ENVIRONMENT_POLICY.md \
  --agents-out ./AGENTS.environment.md \
  --skill-snippet-out ./SKILL.environment.md
```

### Python

macOS / Linux:

```bash
python3 scripts/detect_environment.py --output ./environment-baseline.json
python3 scripts/render_environment_docs.py ./environment-baseline.json \
  --policy-out ./ENVIRONMENT_POLICY.md \
  --agents-out ./AGENTS.environment.md \
  --skill-snippet-out ./SKILL.environment.md
```

Windows example with an explicit Python 3.7+ interpreter:

```powershell
py -3 scripts\detect_environment.py --output .\environment-baseline.json
py -3 scripts\render_environment_docs.py .\environment-baseline.json `
  --policy-out .\ENVIRONMENT_POLICY.md `
  --agents-out .\AGENTS.environment.md `
  --skill-snippet-out .\SKILL.environment.md
```

## Language Control

Both implementations auto-detect locale and choose the document language.

- `zh*` locale -> Chinese output
- `en*` locale -> English output
- unrecognized locale -> English output

Override it explicitly when deterministic output matters:

```bash
node scripts/render_environment_docs.js ./environment-baseline.json --lang zh --stdout policy
node scripts/render_environment_docs.js ./environment-baseline.json --lang en --stdout policy
python3 scripts/render_environment_docs.py ./environment-baseline.json --lang zh --policy-out ./ENVIRONMENT_POLICY.md
python3 scripts/render_environment_docs.py ./environment-baseline.json --lang en --policy-out ./ENVIRONMENT_POLICY.md
```

## Targeted Refresh

Use targeted refresh when a baseline already exists and only one tool or one environment area changed.

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

## Extending Detection

Use a probe file when you need to inspect additional tools or non-default environment elements.
Examples include:

- Docker or Gradle installation checks
- PostgreSQL, Redis, or MySQL service checks
- startup files or project-specific toolchains

See [references/probe-file.md](./references/probe-file.md) for the JSON schema and examples.

## Compatibility And Current Verification

Current verification covered the Windows workflow and confirmed that both implementations run successfully there.

| OS | Script type | Verified | Notes |
| --- | --- | --- | --- |
| Windows | JavaScript | Yes | Verified end-to-end in a Windows environment. |
| Windows | Python | Yes | Verified end-to-end with Python 3.7+ in a Windows environment. |
| macOS | JavaScript | Not yet | Intended support, but no macOS runtime verification was performed. |
| macOS | Python | Not yet | Intended support; requires Python 3.7+. |
| Linux | JavaScript | Yes | Verified end-to-end on Ubuntu 22.04 with Node.js 20. |
| Linux | Python | Yes | Verified end-to-end on Ubuntu 22.04 with Python 3.10. |

## Notes

- This repository no longer ships POSIX shell wrappers because they only forwarded to Python and did not provide a standalone implementation.
- If a Windows machine has multiple Python installs, prefer an explicit Python 3.7+ path instead of assuming `python` points to the right interpreter.
- Older enterprise distributions may need self-managed runtimes when their default repositories cannot provide Python 3.7+ or a modern Node.js release.
- The skill still enforces the same hard rules: do not install, upgrade, uninstall, or silently reconfigure tools without explicit user approval.
