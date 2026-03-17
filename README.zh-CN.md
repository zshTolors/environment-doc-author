# Environment Doc Author

[English](./README.md)

从当前机器的真实环境状态出发，生成并维护环境基线文档。
这个仓库提供两套独立实现：一套纯 Node.js，一套 Python。两者都可以探测当前环境、维护机器可读的 baseline，并渲染 `ENVIRONMENT_POLICY.md`、`AGENTS` 片段和可复用的 skill 片段。

## 实现方式

- `scripts/detect_environment.js` / `scripts/render_environment_docs.js`
  - 纯 Node.js 实现
  - 不依赖 Python
- `scripts/detect_environment.py` / `scripts/render_environment_docs.py`
  - Python 实现
  - 要求 Python 3.7+

## 内置覆盖范围

默认探测器现在已经内置支持主流开发工具链，包括：

- 源码管理与核心运行时
  - Git、Java、`javac`、Maven、Gradle、Node.js、npm、pnpm、Yarn、Python
- 系统与编译型语言工具链
  - Go、Rust（`rustc`、Cargo、Rustup）、Clang、Clang++、GCC、G++、CMake、Make、Ninja
- 其他常见开发工具
  - Ruby、RubyGems、Bundler、PHP、Composer、.NET SDK、Docker、kubectl
- 重要环境变量
  - `JAVA_HOME`、`MAVEN_HOME`、`GRADLE_HOME`、`GOROOT`、`GOPATH`、`CARGO_HOME`、`RUSTUP_HOME`、`NVM_DIR`、`PYENV_ROOT`、`RBENV_ROOT`、`SDKMAN_DIR`、`HOMEBREW_PREFIX`、`VIRTUAL_ENV`、`GEM_HOME`、`BUNDLE_PATH`、`COMPOSER_HOME`、`DOTNET_ROOT`、`CC`、`CXX`

## 输出文件

- `environment-baseline.json`
  - 机器可读的事实基线，记录工具路径、版本、环境变量、PATH 关键项和定向检查结果
- `ENVIRONMENT_POLICY.md`
  - 当前机器的人类可读环境政策
- `AGENTS.environment.md`
  - 供代理使用的简短执行规则
- `SKILL.environment.md`
  - 从同一份 baseline 生成的可复用 skill 片段

## 运行要求

- JavaScript 入口
  - 建议 Node.js 18+
  - 不需要 Python
- Python 入口
  - 要求 Python 3.7+
  - 在 Windows 上，先确认实际调用到的是哪个解释器，再运行 Python 脚本
  - 如果机器上装了多个 Python，优先显式指定 Python 3.7+，不要默认假设 `python` 一定指向正确版本

## 仓库结构

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

## 快速开始

### JavaScript

```bash
node scripts/detect_environment.js --output ./environment-baseline.json
node scripts/render_environment_docs.js ./environment-baseline.json \
  --policy-out ./ENVIRONMENT_POLICY.md \
  --agents-out ./AGENTS.environment.md \
  --skill-snippet-out ./SKILL.environment.md
```

### Python

macOS / Linux：

```bash
python3 scripts/detect_environment.py --output ./environment-baseline.json
python3 scripts/render_environment_docs.py ./environment-baseline.json \
  --policy-out ./ENVIRONMENT_POLICY.md \
  --agents-out ./AGENTS.environment.md \
  --skill-snippet-out ./SKILL.environment.md
```

Windows 示例：

```powershell
py -3 scripts\detect_environment.py --output .\environment-baseline.json
py -3 scripts\render_environment_docs.py .\environment-baseline.json `
  --policy-out .\ENVIRONMENT_POLICY.md `
  --agents-out .\AGENTS.environment.md `
  --skill-snippet-out .\SKILL.environment.md
```

## 语言控制

两套实现都会自动检测 locale 并选择文档语言：

- `zh*` locale -> 输出中文文档
- `en*` locale -> 输出英文文档
- 无法识别的 locale -> 默认输出英文文档

如果你需要确定输出语言，也可以显式覆盖：

```bash
node scripts/render_environment_docs.js ./environment-baseline.json --lang zh --stdout policy
node scripts/render_environment_docs.js ./environment-baseline.json --lang en --stdout policy
python3 scripts/render_environment_docs.py ./environment-baseline.json --lang zh --policy-out ./ENVIRONMENT_POLICY.md
python3 scripts/render_environment_docs.py ./environment-baseline.json --lang en --policy-out ./ENVIRONMENT_POLICY.md
```

## 定向刷新

当 baseline 已存在，而且只有一个工具或一个环境区域发生变化时，使用定向刷新更合适。

JavaScript：

```bash
node scripts/detect_environment.js \
  --baseline ./environment-baseline.json \
  --tools java,maven \
  --output ./environment-baseline.json

node scripts/render_environment_docs.js ./environment-baseline.json \
  --policy-out ./ENVIRONMENT_POLICY.md \
  --agents-out ./AGENTS.environment.md
```

Python：

```bash
python3 scripts/detect_environment.py \
  --baseline ./environment-baseline.json \
  --tools java,maven \
  --output ./environment-baseline.json

python3 scripts/render_environment_docs.py ./environment-baseline.json \
  --policy-out ./ENVIRONMENT_POLICY.md \
  --agents-out ./AGENTS.environment.md
```

## 如何扩展检测

当你需要检测内置探测范围之外的工具或环境要素时，使用 probe file。
常见场景包括：

- 服务级或项目级的额外检查
- PostgreSQL、Redis、MySQL 等服务或端口
- 启动脚本或项目特定工具链

JSON 格式和示例见 [references/probe-file.md](./references/probe-file.md)。

## 兼容性与当前实测情况

当前已完成 Windows、macOS 与 Linux 工作流验证，并确认两套实现都可以在这些已验证路径上正常运行。

| 环境 | 脚本类型 | 是否测通 | 备注 |
| --- | --- | --- | --- |
| Windows | JavaScript | 是 | 已在 Windows 环境完成端到端实测 |
| Windows | Python | 是 | 已在 Windows 环境用 Python 3.7+ 完成端到端实测 |
| macOS | JavaScript | 是 | 已在 2026 年 3 月 17 日使用 macOS 26.2（Apple silicon）和 Node.js 24.14.0 完成端到端实测 |
| macOS | Python | 是 | 已在 2026 年 3 月 17 日使用 macOS 26.2（Apple silicon）和 Python 3.9.6 完成端到端实测 |
| Linux | JavaScript | 是 | 已在 Ubuntu 22.04 上用 Node.js 20 完成端到端实测 |
| Linux | Python | 是 | 已在 Ubuntu 22.04 上用 Python 3.10 完成端到端实测 |

## 说明

- 这个仓库不再提供 POSIX shell 包装脚本，因为它们之前只是转发到 Python，没有独立实现价值
- 如果 Windows 机器上同时存在多个 Python，优先使用明确的 Python 3.7+ 解释器，而不是默认假定 `python` 指向正确版本
- 对较老的企业发行版，如果系统仓库无法提供 Python 3.7+ 或较新的 Node.js，通常需要自行管理运行时
- skill 仍然保留同样的硬规则：未经用户明确同意，不得安装、升级、卸载或静默修改本机环境
