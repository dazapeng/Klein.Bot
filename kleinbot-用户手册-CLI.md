# KleinBot CLI 使用手册（命令行版）

> **版本：** v2.8.1 | **许可证：** Apache-2.0 | **适用平台：** macOS / Linux / Windows  
> **官网：** https://qianpc.com | **GitHub：** https://github.com/kleinbot/kleinbot

---

## 目录

1. [概述](#1-概述)
2. [安装指南](#2-安装指南)
3. [两种运行模式](#3-两种运行模式)
4. [快速入门](#4-快速入门)
5. [核心工作流详解](#5-核心工作流详解)
6. [交互模式详解](#6-交互模式详解)
7. [无头模式详解](#7-无头模式详解)
8. [配置与模型选择](#8-配置与模型选择)
9. [自定义与扩展](#9-自定义与扩展)
10. [MCP 服务器扩展](#10-mcp-服务器扩展)
11. [实战示例](#11-实战示例)
12. [CLI 命令参考](#12-cli-命令参考)
13. [故障排除](#13-故障排除)

---

## 1. 概述

### 1.1 什么是 KleinBot CLI？

KleinBot CLI 是一个**自主编码代理（Autonomous Coding Agent）**，运行在终端中。它能够理解你的自然语言指令，并自动完成以下操作：

- **读取和编辑文件**：扫描项目、创建新文件、修改现有代码
- **执行终端命令**：运行构建、测试、安装依赖等任意命令
- **浏览器自动化**：测试网页、抓取信息、填写表单
- **MCP 服务器扩展**：通过 MCP（Model Context Protocol）连接外部工具和服务
- **支持 30+ AI 模型提供商**：Anthropic、OpenAI、OpenRouter、AWS Bedrock、Google Gemini、DeepSeek、本地模型等

### 1.2 核心能力

| 能力 | 说明 | 带来的好处 |
|------|------|-----------|
| 文件操作 | 读取、创建、编辑项目文件 | 无需手动编写重复代码，AI 帮你完成 |
| 命令执行 | 运行终端命令（需你批准） | 自动化构建、测试、部署流程 |
| 浏览器测试 | 启动浏览器进行 Web 测试 | 前端开发时可自动验证 UI |
| 管道集成 | 支持 `\|` 管道和 `>` 重定向 | 可嵌入 Shell 脚本和 CI/CD 流程 |
| 多模型 | 自由选择底层 AI 模型 | 根据任务复杂度和成本灵活选择 |

### 1.3 与 VS Code 扩展的关系

KleinBot 提供两种使用方式：

- **VS Code/JetBrains 扩展（VSIX）**：在 IDE 内使用，提供图形化界面，适合日常交互式开发
- **CLI 命令行工具**：在终端使用，支持交互和无头两种模式，适合自动化脚本、CI/CD 集成和远程服务器场景

两者共享相同的核心引擎和能力，只是交互方式不同。本手册专门介绍 CLI 版本的使用方法。

---

## 2. 安装指南

### 2.1 前置要求

- **Node.js** ≥ 20.0.0（推荐使用 LTS 版本）
- **操作系统**：macOS、Linux 或 Windows（支持 x64 和 arm64 架构）

验证 Node.js 版本：

```bash
node --version
# 应输出 v20.x.x 或更高版本
```

### 2.2 安装方式

#### 方式一：通过 npm 安装（推荐）

```bash
npm install -g kleinbot
```

安装完成后验证：

```bash
kleinbot --version
# 应输出版本号，如 2.8.1
```

#### 方式二：通过 Homebrew 安装（macOS/Linux）

```bash
brew install kleinbot
```

#### 方式三：手动下载

从 GitHub Releases 页面下载对应平台的二进制包：

```
https://github.com/kleinbot/kleinbot/releases
```

### 2.3 认证授权

首次运行 `kleinbot` 时，系统会引导你完成认证：

1. CLI 会打开浏览器，跳转到 `https://app.qianpc.com` 授权页面
2. 使用你的账户登录并授权
3. 授权完成后，CLI 会自动保存凭据

如果浏览器无法自动打开，CLI 会输出一个 URL，你可以手动复制到浏览器中完成认证。

```bash
# 首次运行
kleinbot
# 按提示完成认证即可
```

### 2.4 升级

```bash
# npm 安装
npm update -g kleinbot

# Homebrew 安装
brew upgrade kleinbot
```

---

## 3. 两种运行模式

KleinBot CLI 有两种运行模式，系统会根据调用方式**自动检测**并选择合适的模式。

### 3.1 交互模式（Interactive Mode）

交互模式提供一个完整的终端聊天界面（TTY），你可以实时与 AI 对话。适用于：

- 日常开发中的即时代码辅助
- 需要多轮对话的复杂任务
- 需要人工批准文件修改和命令执行的场景

**触发方式：**

```bash
# 直接启动
kleinbot

# 带提示词启动（仍然进入交互模式）
kleinbot "帮我创建一个 Express 服务器"
```

### 3.2 无头模式（Headless Mode）

无头模式（也称为非交互模式/自主模式）适合自动化和脚本场景。在无头模式下：

- 不会进入交互界面
- 结果以 JSON 格式输出
- 可通过管道与其他命令组合
- 支持 `-y`（YOLO 模式）自动批准所有操作

**触发方式：**

```bash
# 使用 -y 标志（YOLO 模式，自动批准）
kleinbot -y "修复所有 ESLint 错误"

# 使用 --json 标志（输出 JSON 格式）
kleinbot --json "分析代码质量"

# 通过管道输入
cat issue.txt | kleinbot "分析这个问题"

# 输出重定向到文件
kleinbot "生成报告" > report.txt

# 组合使用
kleinbot -y --json "执行代码审查" > review.json
```

### 3.3 模式自动检测规则

| 调用方式 | 自动选择的模式 |
|----------|---------------|
| `kleinbot` | 交互模式 |
| `kleinbot "提示词"` | 交互模式 |
| `kleinbot -y "提示词"` | 无头模式 |
| `kleinbot --json "提示词"` | 无头模式 |
| `cat file \| kleinbot "提示词"` | 无头模式 |
| `kleinbot "提示词" > output.txt` | 无头模式 |

### 3.4 两种模式的对比

| 特性 | 交互模式 | 无头模式 |
|------|---------|---------|
| 终端界面 | 完整的 TUI（Ink/React） | 纯文本输出 |
| 人工批准 | 默认需要批准 | `-y` 下自动批准 |
| 多轮对话 | 支持 | 单次任务 |
| 输出格式 | 格式化终端输出 | 文本或 JSON |
| 适用场景 | 日常开发、调试 | CI/CD、脚本、自动化 |
| 文件修改 | 每步可审查 | 自动执行 |

---

## 4. 快速入门

### 4.1 5 分钟上手示例

以下示例演示如何用 KleinBot CLI 从零创建一个 Express API 服务。

```bash
# 步骤 1：创建一个新项目目录
mkdir my-api && cd my-api

# 步骤 2：初始化 npm 项目
npm init -y

# 步骤 3：启动 KleinBot（交互模式）
kleinbot
```

在 KleinBot 的对话界面中，输入以下提示词：

```
请帮我创建一个简单的 Express.js API 服务：
1. 安装 express 依赖
2. 创建 src/index.js，实现一个 /api/hello 端点，返回 JSON { "message": "Hello from KleinBot!" }
3. 添加 npm start 脚本
4. 启动服务器让我测试
```

KleinBot 会：

1. 进入**Plan 模式**，分析需求并制定执行计划
2. 向你展示计划，等待确认
3. 切换到**Act 模式**，逐步执行（安装依赖、创建文件、启动服务）
4. 每步操作前会请求你的批准

### 4.2 使用无头模式完成同样任务

如果你希望全自动执行（例如在 CI/CD 中），使用无头模式：

```bash
kleinbot -y "
请帮我创建一个简单的 Express.js API 服务：
1. 安装 express 依赖
2. 创建 src/index.js，实现 /api/hello 端点
3. 添加 npm start 脚本
" --mode act
```

`-y` 标志表示自动批准所有操作，`--mode act` 跳过规划阶段直接执行。

---

## 5. 核心工作流详解

### 5.1 Plan & Act 模式（规划与执行）

这是 KleinBot 最核心的工作机制。每个任务都会经历两个阶段：

#### Plan（规划阶段）

KleinBot 先分析你的需求，制定详细计划：

- 拆解任务步骤
- 确定需要哪些文件、哪些命令
- 向你展示完整计划
- 你可以确认、修改或取消

**好处：** 在 AI 动手之前，你有机会审查方案，避免错误的方向。

#### Act（执行阶段）

确认计划后，KleinBot 开始执行：

- 逐步执行计划的每一步
- 创建/修改文件
- 运行必要的命令
- 向你报告执行结果

#### 快捷键控制

| 快捷键 | 操作 |
|--------|------|
| `Tab` | 切换到 Plan 模式 |
| `Shift+Tab` | 切换到 Act 模式并启用自动批准 |

### 5.2 任务管理

KleinBot 将所有对话历史保存为**任务**，你可以随时查看和恢复。

```bash
# 查看任务列表（交互模式下使用 / 命令）
/tasks list

# 恢复之前的任务
/tasks resume <task-id>

# 删除任务
/tasks delete <task-id>

# 查看当前任务详情
/tasks current
```

**好处：** 每个任务都是独立的上下文，不会互相污染。你可以同时处理多个项目或功能。

### 5.3 文件操作

在交互模式中，你可以使用 `@` 符号引用文件：

```
请分析 @src/server.js 中的错误处理逻辑，并提出改进建议
```

KleinBot 可以：

- **读取文件**：分析代码、理解结构
- **创建文件**：生成新代码文件
- **编辑文件**：精确修改指定行或函数
- **搜索代码**：在整个项目中搜索特定模式

**好处：** 你不需要手动打开文件、复制粘贴代码，只需用 `@` 引用即可。

### 5.4 命令执行

KleinBot 可以执行终端命令，但每次都需要你的批准（除非使用 `-y` 模式）：

```
请运行单元测试并告诉我哪些失败了
```

KleinBot 会：

1. 提出要执行的命令（如 `npm test`）
2. 等待你批准
3. 执行命令并解析结果
4. 根据结果进行后续操作

**好处：** AI 可以帮你运行命令，但你始终掌握最终控制权。

### 5.5 检查点系统（Checkpoints）

每次文件修改前，KleinBot 自动创建检查点：

- **自动保存**：修改文件前自动保存当前状态
- **随时回退**：如果结果不满意，可以回退到之前的检查点
- **跨会话持久化**：检查点在任务结束后仍然保留

**如何使用：**

```
# 在交互模式中
/checkpoints list          # 查看所有检查点
/checkpoints restore <id>  # 回退到指定检查点
```

**好处：** 即使 AI 做出了你不满意的修改，一键即可恢复，零风险尝试。

---

## 6. 交互模式详解

### 6.1 启动交互模式

```bash
# 直接启动
kleinbot

# 带初始提示词
kleinbot "优化这个项目的性能"
```

### 6.2 终端界面

交互模式使用基于 React 19 和 Ink 构建的终端 UI，提供：

- **多行输入框**：支持复杂的长文本提示
- **语法高亮**：代码块自动高亮显示
- **文件差异预览**：修改前后对比（类似 `git diff`）
- **命令执行状态**：实时显示命令执行进度

### 6.3 斜杠命令（Slash Commands）

在交互模式的输入框中，输入 `/` 可以触发以下命令：

| 命令 | 功能 | 示例 |
|------|------|------|
| `/help` | 显示帮助信息 | `/help` |
| `/clear` | 清空当前对话 | `/clear` |
| `/compact` | 压缩上下文（释放 token） | `/compact` |
| `/tasks list` | 列出所有任务 | `/tasks list` |
| `/tasks resume` | 恢复指定任务 | `/tasks resume abc123` |
| `/tasks delete` | 删除指定任务 | `/tasks delete abc123` |
| `/checkpoints list` | 列出检查点 | `/checkpoints list` |
| `/checkpoints restore` | 回退到检查点 | `/checkpoints restore 1` |
| `/model` | 切换 AI 模型 | `/model claude-sonnet-4-20250514` |
| `/config` | 查看当前配置 | `/config` |
| `/exit` | 退出 KleinBot | `/exit` |
| `/mcp` | 管理 MCP 服务器 | `/mcp list` |
| `/workspace` | 管理工作区文件 | `/workspace add ./src` |

### 6.4 文件引用（@ Mentions）

使用 `@` 符号引用项目中的文件或目录：

```
请重构 @src/utils/helpers.js ，提取出通用的工具函数到 @src/utils/common.js
```

支持：

- `@filename` — 引用单个文件
- `@directory/` — 引用整个目录
- `@filename:10-20` — 引用文件的指定行范围

**好处：** 无需手动打开文件复制粘贴，AI 自动读取相关文件内容。

### 6.5 对话历史管理

- **滚动查看**：使用方向键或 `PgUp`/`PgDn` 浏览历史消息
- **上下文持久化**：退出后重新启动，可以恢复之前的对话
- **自动 compact**：当上下文接近 token 上限时，自动压缩优化

---

## 7. 无头模式详解

### 7.1 何时使用无头模式

无头模式适用于以下场景：

- **CI/CD 管道**：在 GitHub Actions、Jenkins 中自动执行任务
- **自动化脚本**：定期运行的代码质量检查、文档生成
- **批量处理**：对多个项目执行相同的操作
- **与其他工具集成**：通过管道连接 grep、jq 等命令行工具

### 7.2 基本语法

```bash
kleinbot [选项] "你的提示词"
```

#### 常用选项

| 选项 | 说明 | 示例 |
|------|------|------|
| `-y, --yolo` | 自动批准所有操作 | `kleinbot -y "修复 lint 错误"` |
| `--json` | 输出 JSON 格式结果 | `kleinbot --json "代码审查"` |
| `--mode act` | 跳过 Plan 规划直接执行 | `kleinbot -y --mode act "创建单元测试"` |
| `--thinking` | 启用/禁用扩展思考 | `kleinbot --thinking on "复杂算法分析"` |
| `--config` | 指定配置文件名 | `kleinbot --config ci-config "CI 任务"` |
| `--cwd` | 指定工作目录 | `kleinbot --cwd /path/to/project "分析代码"` |

### 7.3 输出格式

#### 默认输出（纯文本）

```bash
kleinbot "统计 src/ 目录下所有 JS 文件的行数"
```

#### JSON 输出

```bash
kleinbot --json "分析代码复杂度" > analysis.json
```

JSON 输出结构：

```json
{
  "result": "任务执行结果文本",
  "files_changed": ["src/index.js", "src/utils.js"],
  "commands_run": ["npm test"],
  "exit_code": 0,
  "tokens_used": 15420,
  "duration_ms": 32000
}
```

### 7.4 管道与重定向

KleinBot CLI 完全支持 Unix 管道哲学：

```bash
# 将代码审查结果输出到文件
kleinbot "审查 src/ 的安全问题" > security-review.md

# 将 issue 内容通过管道传入
gh issue view 42 --json body | kleinbot "分析这个 issue 的根因"

# 组合 jq 进行后处理
kleinbot -y --json "代码质量报告" | jq '.result'
```

### 7.5 退出码

| 退出码 | 含义 |
|--------|------|
| 0 | 任务成功完成 |
| 1 | 任务执行中出现错误 |
| 2 | 配置错误（如 API 密钥无效） |
| 130 | 用户中断（SIGINT） |

---

## 8. 配置与模型选择

### 8.1 配置文件位置

CLI 的配置文件位于：

- **macOS/Linux**：`~/.config/kleinbot/config.json`
- **Windows**：`%APPDATA%\kleinbot\config.json`

或项目级别的配置：

- `<项目根目录>/.kleinbot/config.json`

### 8.2 基本配置示例

```json
{
  "defaultModel": "claude-sonnet-4-20250514",
  "models": {
    "claude-sonnet-4-20250514": {
      "provider": "anthropic",
      "apiKey": "sk-ant-xxx",
      "maxTokens": 200000,
      "thinking": true
    }
  },
  "approval": {
    "autoApprove": false,
    "maxAutoApproveCost": 0.50
  },
  "workspace": {
    "autoLoadPatterns": ["src/**/*.ts"]
  }
}
```

### 8.3 支持的 AI 提供商

KleinBot 支持 **30+ 云提供商**和本地模型：

| 提供商 | 配置标识 | 特点 |
|--------|---------|------|
| Anthropic | `anthropic` | Claude 系列，代码能力最强 |
| OpenAI | `openai` | GPT-4o 系列 |
| OpenRouter | `openrouter` | 统一 API 访问多种模型 |
| AWS Bedrock | `aws-bedrock` | Amazon 托管，企业友好 |
| Google Gemini | `google-gemini` | Gemini 2.5 Pro |
| DeepSeek | `deepseek` | 高性价比 |
| Qwen（通义千问） | `qwen` | 阿里云 |
| Moonshot（月之暗面） | `moonshot` | Kimi |
| Doubao（豆包） | `doubao` | 字节跳动 |
| Minimax | `minimax` | 海螺 AI |
| 华为云 Maas | `huawei-cloud-maas` | 华为盘古 |
| Groq | `groq` | 超快推理速度 |
| Cerebras | `cerebras` | 高速推理 |
| Mistral AI | `mistral-ai` | 欧洲领先 |
| Ollama | `ollama` | 本地运行开源模型 |
| LM Studio | `lm-studio` | 桌面端本地模型 |

### 8.4 本地模型配置

#### Ollama

```bash
# 安装并启动 Ollama
brew install ollama
ollama serve

# 拉取模型
ollama pull qwen2.5-coder:14b

# 在 KleinBot 中配置
kleinbot
# 交互模式中使用 /model 切换，或修改 config.json
```

配置示例：

```json
{
  "models": {
    "local-qwen": {
      "provider": "ollama",
      "model": "qwen2.5-coder:14b",
      "baseUrl": "http://localhost:11434"
    }
  }
}
```

**好处：** 数据不出本地，零网络依赖，零 API 费用。

### 8.5 上下文窗口说明

不同模型有不同的上下文窗口（即一次能处理的 token 量）：

| 模型 | 上下文窗口 | 适合场景 |
|------|-----------|---------|
| Claude Sonnet 4 | 200K tokens | 大型项目、多文件操作 |
| GPT-4o | 128K tokens | 常规开发任务 |
| DeepSeek V3 | 128K tokens | 高性价比选择 |
| Gemini 2.5 Pro | 1M+ tokens | 超大型代码库分析 |
| Qwen 2.5 Coder 32B | 128K tokens | 本地代码生成 |

### 8.6 模型选择建议

| 场景 | 推荐模型 | 原因 |
|------|---------|------|
| 日常编码 | Claude Sonnet 4 | 代码质量最佳 |
| 大型重构 | Claude Sonnet 4 + 扩展思考 | 深度推理能力 |
| 预算有限 | DeepSeek V3 | 性价比极高 |
| 隐私敏感 | Ollama + Qwen 2.5 Coder | 完全本地化 |
| 超大型项目分析 | Gemini 2.5 Pro | 1M token 上下文 |
| 快速原型 | Groq + Llama 3.3 | 推理速度最快 |

---

## 9. 自定义与扩展

### 9.1 KleinBot 规则（Rules）

规则是持久化的指令，可以定义 AI 的行为规范。规则文件使用 Markdown 格式，存放在 `.kleinbotrules/` 目录下。

#### 创建规则

在项目根目录创建 `.kleinbotrules/` 目录，然后添加 Markdown 文件：

```
.kleinbotrules/
├── coding-standards.md      # 代码规范
├── api-design.md            # API 设计规范
└── testing.md               # 测试要求
```

**规则示例**（`.kleinbotrules/coding-standards.md`）：

```markdown
# 项目编码规范

- 使用 TypeScript 编写所有新代码
- 遵循 ESLint 配置的规则
- 每个函数必须有 JSDoc 注释
- 使用 async/await 而非 Promise.then()
- 变量命名使用 camelCase
- 文件命名使用 kebab-case
```

#### 条件规则

规则可以通过 YAML frontmatter 设置生效条件：

```markdown
---
paths:
  - "src/api/**"
  - "src/services/**"
---
# API 服务层规范

- 所有 API 端点必须有输入验证
- 使用 HTTP 状态码标准
- 错误响应格式统一为 { error: string, details?: any }
```

**好处：** 确保 AI 始终遵循团队规范，生成的代码与项目风格一致。

#### 规则优先级

1. 项目级 `.kleinbotrules/`（优先级最高）
2. 用户全局规则
3. 内置默认规则（优先级最低）

### 9.2 技能系统（Skills）

技能是按需加载的专业模块，包含特定领域的知识和指令。与规则不同，技能不会被自动加载，需要明确调用。

#### 技能结构

每个技能是一个包含 `SKILL.md` 的目录：

```
skills/
└── database-migration/
    ├── SKILL.md              # 技能定义（必需）
    ├── docs/
    │   └── migration-guide.md # 参考文档
    ├── templates/
    │   └── migration.tmpl     # 模板文件
    └── scripts/
        └── migrate.sh         # 工具脚本
```

**SKILL.md 示例**：

```markdown
---
name: database-migration
description: 数据库迁移专家，帮助生成和执行数据库迁移
triggers:
  - 数据库迁移
  - schema change
  - migration
---

# 数据库迁移技能

当用户请求数据库相关的迁移操作时，按以下流程执行：

1. 分析现有数据库 schema
2. 生成迁移脚本（up 和 down）
3. 提供回滚方案
4. 执行迁移并验证

## 支持的数据库

- PostgreSQL
- MySQL
- SQLite
```

**好处：** 将团队最佳实践固化为可复用的技能模块，新人也能获得专家级指导。

### 9.3 工作流（Workflows）

工作流是步骤化的自动化流程，通过 Markdown 文件定义，使用 `/filename.md` 调用。

#### 创建工作流

在 `.kleinbot/workflows/` 目录下创建 Markdown 文件：

```markdown
# 发布新版本工作流

## 步骤 1：检查状态
运行 `git status` 确保工作区干净

## 步骤 2：更新版本号
修改 package.json 中的 version 字段

## 步骤 3：生成 CHANGELOG
分析最近的 git log，生成 CHANGELOG 条目

## 步骤 4：运行测试
执行 `npm test` 确保所有测试通过

## 步骤 5：提交并打标签
提交修改并创建 git tag
```

#### 使用工作流

```bash
# 在交互模式中
/release-new-version
```

KleinBot 可以直接从完成的任务自动生成工作流模板，方便复用。

**好处：** 将重复性任务标准化为工作流，减少人为遗漏，提高效率。

### 9.4 Hooks 生命周期钩子

Hooks 是在任务生命周期的特定时刻执行的脚本，可以实现自动化检查和控制。

#### 8 个 Hook 点

| Hook 点 | 触发时机 | 典型用途 |
|----------|---------|---------|
| `TaskStart` | 任务开始时 | 加载项目上下文 |
| `TaskResume` | 任务恢复时 | 恢复之前的状态 |
| `TaskComplete` | 任务完成时 | 发送通知、记录日志 |
| `TaskCancel` | 任务取消时 | 清理临时文件 |
| `PreToolUse` | 工具使用前 | 审批、日志记录 |
| `PostToolUse` | 工具使用后 | 结果验证、通知 |
| `UserPromptSubmit` | 用户提交提示前 | 提示增强、注入上下文 |
| `PreCompact` | 上下文压缩前 | 保存关键信息 |

#### 配置方式

在配置文件或 `.kleinbot/hooks/` 目录下创建脚本：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "command": "npm run lint -- ${KLEINBOT_FILE_PATH}"
      }
    ],
    "TaskComplete": [
      {
        "command": "terminal-notifier -title 'KleinBot' -message '任务完成'"
      }
    ]
  }
}
```

**好处：** 自动化质量检查、合规审查、通知等流程，在 AI 操作后即时反馈。

### 9.5 .kleinbotignore 文件

类似于 `.gitignore`，`.kleinbotignore` 文件告诉 KleinBot 忽略某些文件或目录：

```
# 依赖目录
node_modules/
vendor/
.pytest_cache/

# 构建产物
dist/
build/
*.min.js

# 大文件
*.zip
*.tar.gz
*.mp4

# 密钥文件
.env
*.pem
```

**好处：** 减少无关文件对上下文的占用，让 AI 专注于项目代码。

### 9.6 仓库档案（Repo Profile）

通过 `.kleinbot/repo-profile.md` 文件，你可以向 KleinBot 介绍项目的整体情况：

```markdown
# 项目概况

## 技术栈
- 前端：React 18 + TypeScript + Tailwind CSS
- 后端：Node.js + Express + PostgreSQL
- 测试：Vitest + Playwright

## 项目结构
- `src/client/` — 前端代码
- `src/server/` — 后端 API
- `tests/` — 测试文件

## 关键约定
- API 路由使用 RESTful 风格
- 数据库迁移使用 Knex.js
- 所有 PR 需要通过 CI 检查
```

**好处：** 每次开始任务时，AI 自动理解项目背景，减少重复解释。

---

## 10. MCP 服务器扩展

### 10.1 什么是 MCP？

MCP（Model Context Protocol）是一种开放协议，允许 AI 代理连接外部工具和数据源。通过 MCP 服务器，KleinBot 可以：

- 访问数据库（PostgreSQL、SQLite 等）
- 调用外部 API（GitHub、Slack、Jira）
- 读取文档和知识库
- 执行专门的工具（如代码搜索、图片处理）

### 10.2 安装 MCP 服务器

#### 从 MCP 市场安装

```bash
# 在交互模式中
/mcp marketplace
# 浏览并选择需要的服务器，KleinBot 自动完成安装和配置
```

#### 手动添加 MCP 服务器

编辑配置文件，在 `mcpServers` 中添加：

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_xxx"
      }
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "args": ["postgresql://localhost/mydb"]
    }
  }
}
```

### 10.3 管理 MCP 服务器

```bash
# 列出所有已安装的服务器
/mcp list

# 启用/禁用服务器
/mcp enable github
/mcp disable github

# 重启服务器
/mcp restart github

# 删除服务器
/mcp remove postgres
```

### 10.4 使用 MCP 工具

安装后的 MCP 工具会自动对 KleinBot 可用。你可以在提示词中明确指定：

```
请使用 GitHub MCP 工具，帮我创建一个 issue，标题为"修复登录页面的 Bug"
```

或者在描述任务时，KleinBot 会自动判断是否需要调用 MCP 工具：

```
请检查 PostgreSQL 数据库中 users 表的结构，并帮我生成对应的 TypeScript 类型定义
```

### 10.5 MCP 传输机制

| 传输方式 | 说明 | 适用场景 |
|---------|------|---------|
| STDIO | 标准输入输出，进程内通信 | 本地服务器 |
| SSE（Server-Sent Events） | HTTP 流式通信 | 远程服务器 |
| Streamable HTTP | 可流式传输的 HTTP | 需要代理的场景 |

**好处：** 通过 MCP 扩展生态，KleinBot 的能力可以无限扩展，连接任何外部工具和数据源。

---

## 11. 实战示例

### 11.1 GitHub Issue 根因分析

以下脚本自动分析 GitHub Issue 并生成根因分析报告：

```bash
#!/bin/bash
# analyze-issue.sh - 分析 GitHub Issue

ISSUE_URL="$1"
PROMPT="${2:-请分析这个 issue 的根本原因，并提供修复建议}"

kleinbot -y --json --mode act "$PROMPT: $ISSUE_URL" \
  | jq -r '.result' \
  | tee rca-report.md
```

使用方式：

```bash
./analyze-issue.sh "https://github.com/user/repo/issues/42"
```

### 11.2 GitHub PR 自动审查

```bash
#!/bin/bash
# pr-review.sh - 自动审查 PR

PR_URL="$1"

kleinbot -y --json --mode act "
请审查这个 Pull Request：
$PR_URL

审查要点：
1. 代码质量和可读性
2. 潜在的安全问题
3. 性能影响
4. 测试覆盖
5. 是否遵循项目编码规范

请以 Markdown 格式输出审查报告。
" | jq -r '.result' > pr-review.md
```

### 11.3 多模型编排策略

针对不同任务使用不同模型，优化成本和效果：

```bash
# 策略 1：快速任务用便宜模型
kleinbot --config fast-model -y "修复 ESLint 警告"

# 策略 2：复杂审查用强大模型
kleinbot --config review-model --thinking on "深度代码审查"

# 策略 3：多模型共识
# 用两个模型独立分析，比较结果
kleinbot --config model-a --json "分析安全性" > result-a.json
kleinbot --config model-b --json "分析安全性" > result-b.json
diff <(jq -r '.result' result-a.json) <(jq -r '.result' result-b.json)
```

### 11.4 CI/CD 集成示例（GitHub Actions）

```yaml
name: AI Code Review
on: [pull_request]

jobs:
  ai-review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install KleinBot CLI
        run: npm install -g kleinbot

      - name: Run AI Code Review
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          git diff origin/main...HEAD > pr.diff
          kleinbot -y --json --mode act "
          审查以下 PR 变更：
          $(cat pr.diff)

          请提供：
          1. 代码质量评估
          2. 潜在问题
          3. 改进建议
          " | jq -r '.result' > review.md

      - name: Post Review Comment
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const review = fs.readFileSync('review.md', 'utf8');
            await github.rest.issues.createComment({
              ...context.repo,
              issue_number: context.issue.number,
              body: review
            });
```

### 11.5 管道组合示例

```bash
# 查找所有 TODO 注释并生成修复计划
grep -rn "TODO" src/ | kleinbot "分析这些 TODO 的优先级并生成修复计划"

# 代码统计 + AI 分析
cloc src/ --json | kleinbot "分析这个项目的代码组成，识别需要重构的模块"

# 测试失败分析
npm test 2>&1 | kleinbot "分析测试失败原因并提供修复方案"
```

### 11.6 Git Worktree 并行工作

```bash
# 为不同任务创建独立的工作树
git worktree add ../feature-a feature-a-branch
git worktree add ../feature-b feature-b-branch

# 同时在不同工作树中运行 KleinBot
kleinbot --cwd ../feature-a "实现用户认证模块" &
kleinbot --cwd ../feature-b "优化数据库查询性能" &
wait
```

---

## 12. CLI 命令参考

### 12.1 基础命令

```bash
# 启动交互模式
kleinbot

# 带提示词启动
kleinbot "你的任务描述"

# 查看版本
kleinbot --version

# 查看帮助
kleinbot --help
```

### 12.2 无头模式命令

```bash
# 自动批准模式
kleinbot -y "任务描述"

# JSON 输出
kleinbot --json "任务描述"

# 指定运行模式
kleinbot --mode plan "先规划"       # 仅规划
kleinbot --mode act "直接执行"       # 跳过规划

# 启用扩展思考
kleinbot --thinking on "复杂任务"
kleinbot --thinking off "简单任务"

# 指定配置
kleinbot --config ci-config "CI 任务"

# 指定工作目录
kleinbot --cwd /path/to/project "任务描述"

# 使用特定模型
kleinbot --model deepseek-v3 "任务描述"
```

### 12.3 完整选项列表

| 选项 | 简写 | 说明 | 默认值 |
|------|------|------|--------|
| `--yolo` | `-y` | 自动批准所有操作 | `false` |
| `--json` | — | 以 JSON 格式输出 | `false` |
| `--mode` | — | 运行模式：plan/act | `auto` |
| `--thinking` | — | 扩展思考开关 | `auto` |
| `--config` | — | 配置文件名称 | `default` |
| `--model` | — | 指定 AI 模型 | 配置文件默认值 |
| `--cwd` | — | 工作目录路径 | 当前目录 |
| `--verbose` | `-v` | 详细日志输出 | `false` |
| `--version` | `-V` | 显示版本号 | — |
| `--help` | `-h` | 显示帮助信息 | — |

---

## 13. 故障排除

### 13.1 终端问题修复

#### 问题：命令输出显示异常

**解决方案：** 使用后台执行模式

```bash
# 在配置文件或环境变量中设置
export KLEINBOT_BACKGROUND_EXEC=true
```

#### 问题：macOS + Oh-My-Zsh 下终端卡住

**解决方案：** 切换到 bash 或在 zsh 配置中禁用特定插件

```bash
# 临时使用 bash
bash -c "kleinbot '你的任务'"

# 或永久修复：在 ~/.zshrc 中注释掉可能有冲突的插件
```

#### 问题：Windows PowerShell 乱码

**解决方案：** 设置 UTF-8 编码

```powershell
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8
$env:KLEINBOT_UTF8 = "true"
kleinbot "你的任务"
```

### 13.2 网络与代理配置

#### 配置 HTTP 代理

```bash
# CLI 使用环境变量
export https_proxy="http://proxy.company.com:8080"
export http_proxy="http://proxy.company.com:8080"

# 带认证的代理
export https_proxy="http://user:password@proxy.company.com:8080"
```

#### 配置自定义 CA 证书

```bash
export NODE_EXTRA_CA_CERTS="/path/to/custom-ca.crt"
```

#### 验证网络连接

```bash
# 测试 API 连接
curl -I https://api.anthropic.com

# 测试代理连接
curl -I --proxy $https_proxy https://api.anthropic.com
```

### 13.3 任务历史恢复

#### 查看历史记录

```bash
# 在交互模式中
/tasks list
```

#### 恢复丢失的任务

```bash
# 在交互模式中，使用命令面板
/tasks recover

# 或者手动从备份恢复
cp ~/.config/kleinbot/backups/tasks/task-xxx.json ~/.config/kleinbot/tasks/
```

#### 跨机器迁移任务

```bash
# 在源机器上备份
tar -czf kleinbot-backup.tar.gz ~/.config/kleinbot/tasks/

# 在目标机器上恢复
tar -xzf kleinbot-backup.tar.gz -C ~/.config/kleinbot/
```

### 13.4 常见错误码

| 错误 | 原因 | 解决方案 |
|------|------|---------|
| `Authentication failed` | API 密钥无效或过期 | 重新运行 `kleinbot` 完成认证 |
| `Rate limit exceeded` | API 调用频率过高 | 等待一段时间后重试 |
| `Context window exceeded` | 输入内容超过模型上限 | 使用 `/compact` 压缩上下文 |
| `Command not found: kleinbot` | 未安装或 PATH 配置错误 | 重新安装 `npm install -g kleinbot` |
| `Node.js version not supported` | Node.js 版本过低 | 升级到 Node.js ≥ 20 |
| `Network timeout` | 网络连接超时 | 检查代理配置或网络连接 |

---

## 附录 A：快速参考卡片

```
┌─────────────────────────────────────────────────────────┐
│                  KleinBot CLI 快速参考                    │
├─────────────────────────────────────────────────────────┤
│  启动交互模式：          kleinbot                         │
│  启动无头模式：          kleinbot -y "任务"               │
│  JSON 输出：            kleinbot --json "任务"            │
│  查看帮助：             kleinbot --help                   │
│  切换 Plan/Act：        Tab / Shift+Tab                   │
│  引用文件：             @filename                         │
│  斜杠命令：             /help /clear /tasks /checkpoints  │
│  压缩上下文：           /compact                          │
│  退出：                 /exit                             │
├─────────────────────────────────────────────────────────┤
│  管道输入：             cat file | kleinbot "任务"        │
│  输出重定向：           kleinbot "任务" > output.txt      │
│  指定工作目录：         kleinbot --cwd /path "任务"       │
│  指定模型：             kleinbot --model model-id "任务"  │
└─────────────────────────────────────────────────────────┘
```

---

> **提示：** 更多详细信息和最新功能，请访问 [https://qianpc.com](https://qianpc.com) 或查阅 [GitHub 仓库](https://github.com/kleinbot/kleinbot)。
>
> 如需 VS Code/JetBrains 扩展的使用说明，请参阅《KleinBot 用户手册 - VSIX》。