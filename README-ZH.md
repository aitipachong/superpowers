# Superpowers

Superpowers 是一套完整的软件开发方法论，专为你的编程智能体而设计。它构建在一组可组合的技能之上，并包含一些初始指令，确保你的智能体能够正确使用这些技能。


## 我们在招聘！

我们正在招聘一位全职协助 Superpowers 社区和代码工作的人。
你可以在 [这里](https://primeradiant.com/jobs/superpowers-community-engineer/) 了解这份工作的详情。
如果你认识这样的人，请一定要介绍给我们！

## 快速开始

为你的智能体赋予 Superpowers：[Claude Code](#claude-code)、[Antigravity](#antigravity)、[Codex App](#codex-app)、[Codex CLI](#codex-cli)、[Cursor](#cursor)、[Factory Droid](#factory-droid)、[GitHub Copilot CLI](#github-copilot-cli)、[Kimi Code](#kimi-code)、[OpenCode](#opencode)、[Pi](#pi)。

## 工作原理

从你启动编程智能体的那一刻起，它就开始工作了。当智能体看到你正在构建某个东西时，它**不会**立刻开始写代码。相反，它会退后一步，询问你真正想要做什么。

一旦它从对话中梳理出了一份规格说明，就会以简短的片段形式展示给你，方便你实际阅读和消化。

在你确认设计之后，你的智能体会制定一份清晰的实施计划，详细到足以让一位热情但品味差、缺乏判断力、不了解项目背景且厌恶测试的初级工程师也能遵循。它强调真正的红/绿测试驱动开发（TDD）、YAGNI（你不会需要它）原则和 DRY（不要重复自己）原则。

接下来，一旦你说了"开始"，它就会启动一个**子智能体驱动开发**（subagent-driven-development）流程，让智能体逐个完成每个工程任务，检查并审查它们的工作，然后继续前进。你的智能体通常可以自主工作一两个小时，而不会偏离你们一起制定的计划。

除此之外还有很多内容，但这正是该系统的核心。而且由于技能会自动触发，你不需要做任何特殊操作。你的编程智能体就拥有了 Superpowers。

## 商业服务

如果你在大型企业中使用 Superpowers，并需要商业支持、额外的工具或托管支出，请随时发送邮件至 sales@primeradiant.com 联系我们。

## 安装

安装方式因不同的工具（harness）而异。如果你使用多个工具，请为每个工具单独安装 Superpowers。

### Claude Code

Superpowers 可通过 [Claude 官方插件市场](https://claude.com/plugins/superpowers) 获取。

#### 官方市场

- 从 Anthropic 官方市场安装插件：

  ```bash
  /plugin install superpowers@claude-plugins-official
  ```

#### Superpowers 市场

Superpowers 市场提供 Superpowers 及其他一些相关插件。

- 注册市场：

  ```bash
  /plugin marketplace add obra/superpowers-marketplace
  ```

- 从该市场安装插件：

  ```bash
  /plugin install superpowers@superpowers-marketplace
  ```

### Antigravity

从本仓库将 Superpowers 作为插件安装：

```bash
agy plugin install https://github.com/obra/superpowers
```

Antigravity 会运行插件的 session-start 钩子，因此 Superpowers 从第一条消息开始就会生效。使用相同的命令重新安装即可更新。

### Codex App

Superpowers 可通过 [Codex 官方插件市场](https://github.com/openai/plugins) 获取。

- 在 Codex 应用中，点击侧边栏中的插件（Plugins）。
- 你应该会在编程（Coding）部分看到 `Superpowers`。
- 点击 Superpowers 旁边的 `+` 并按照提示操作。

### Codex CLI

Superpowers 可通过 [Codex 官方插件市场](https://github.com/openai/plugins) 获取。

- 打开插件搜索界面：

  ```bash
  /plugins
  ```

- 搜索 Superpowers：

  ```bash
  superpowers
  ```

- 选择 `Install Plugin`（安装插件）。

### Cursor

- 在 Cursor Agent 聊天中，从市场安装：

  ```text
  /add-plugin superpowers
  ```

- 或者在插件市场中搜索 "superpowers"。

### Factory Droid

- 注册市场：

  ```bash
  droid plugin marketplace add https://github.com/obra/superpowers
  ```

- 安装插件：

  ```bash
  droid plugin install superpowers@superpowers
  ```

### GitHub Copilot CLI

- 注册市场：

  ```bash
  copilot plugin marketplace add obra/superpowers-marketplace
  ```

- 安装插件：

  ```bash
  copilot plugin install superpowers@superpowers-marketplace
  ```

### Kimi Code

Superpowers 可在 Kimi Code 的插件市场中获取。

- 打开 Kimi Code 的插件管理器：

  ```text
  /plugins
  ```

- 进入 `Marketplace` > `Superpowers` 并安装它。

- 或者直接从本仓库安装：

  ```text
  /plugins install https://github.com/obra/superpowers
  ```

- 详细文档：[docs/README.kimi.md](docs/README.kimi.md)

### OpenCode

OpenCode 使用自己的插件安装方式；即使你已经在其他工具中使用了 Superpowers，也需要单独安装。

- 告诉 OpenCode：

  ```
  Fetch and follow instructions from https://raw.githubusercontent.com/obra/superpowers/refs/heads/main/.opencode/INSTALL.md
  ```

- 详细文档：[docs/README.opencode.md](docs/README.opencode.md)

### Pi

从本仓库将 Superpowers 作为 Pi 包安装：

```bash
pi install git:github.com/obra/superpowers
```

对于本地开发，使用以下命令运行 Pi，将此 checkout 加载为临时包：

```bash
pi -e /path/to/superpowers
```

Pi 包会加载 Superpowers 技能以及一个小的扩展，该扩展在会话启动时和压缩（compaction）后注入 `using-superpowers` 引导程序。Pi 拥有原生技能，因此不需要兼容性的 `Skill` 工具。子智能体和任务列表工具仍然是可选的 Pi 配套包。

## 基本工作流程

1. **brainstorming（头脑风暴）** - 在编写代码之前激活。通过提问来完善粗略的想法，探索替代方案，将设计分段展示以供验证。保存设计文档。

2. **using-git-worktrees（使用 Git 工作树）** - 在设计确认后激活。在新建分支上创建隔离的工作空间，运行项目设置，验证干净的测试基线。

3. **writing-plans（编写计划）** - 随确认的设计一起激活。将工作分解为小块任务（每个 2-5 分钟）。每个任务都包含确切的文件路径、完整的代码和验证步骤。

4. **subagent-driven-development（子智能体驱动开发）** 或 **executing-plans（执行计划）** - 随计划一起激活。为每个任务分派新的子智能体，进行两阶段审查（规格合规性，然后是代码质量），或以批次方式执行并设置人工检查点。

5. **test-driven-development（测试驱动开发）** - 在实现期间激活。强制执行 RED-GREEN-REFACTOR（红-绿-重构）：编写失败的测试，观察它失败，编写最少量的代码，观察它通过，然后提交。删除在测试之前编写的代码。

6. **requesting-code-review（请求代码审查）** - 在任务之间激活。对照计划进行审查，按严重程度报告问题。严重问题会阻止进度推进。

7. **finishing-a-development-branch（完成开发分支）** - 在任务完成时激活。验证测试，呈现选项（合并/PR/保留/丢弃），清理工作树。

**智能体在执行任何任务之前都会检查相关的技能。** 这是强制性的工作流程，而非建议。

## 内容概览

### 技能库

**测试**
- **test-driven-development** - 红-绿-重构循环（包含测试反模式参考）

**调试**
- **systematic-debugging** - 四阶段根因分析流程（包含根因追踪、纵深防御、基于条件的等待技术）
- **verification-before-completion** - 确保问题确实已修复

**协作**
- **brainstorming** - 苏格拉底式设计提炼
- **writing-plans** - 详细的实施计划
- **executing-plans** - 带检查点的批量执行
- **dispatching-parallel-agents** - 并发子智能体工作流
- **requesting-code-review** - 审查前检查清单
- **receiving-code-review** - 响应反馈
- **using-git-worktrees** - 并行开发分支
- **finishing-a-development-branch** - 合并/PR 决策工作流
- **subagent-driven-development** - 快速迭代，包含两阶段审查（规格合规性，然后是代码质量）

**元**
- **writing-skills** - 遵循最佳实践创建新技能（包含测试方法论）
- **using-superpowers** - 技能系统介绍

## 理念

- **测试驱动开发** - 始终先写测试
- **系统化优于临时性** - 流程优于猜测
- **降低复杂性** - 简洁是首要目标
- **证据胜于声明** - 在宣布成功之前先验证

阅读[最初的发布声明](https://blog.fsck.com/2025/10/09/superpowers/)。

## 贡献

Superpowers 的一般贡献流程如下。请记住，我们通常不接受新技能的贡献，并且任何技能的更新必须适用于我们支持的所有编程智能体。

1. Fork 本仓库
2. 切换到 `dev` 分支
3. 为你的工作创建一个分支
4. 遵循 `writing-skills` 技能来创建和测试新的以及修改后的技能
5. 提交 PR，并确保填写 PR 模板

技能行为测试使用来自 [superpowers-evals](https://github.com/prime-radiant-inc/superpowers-evals/) 的 drill eval 工具，克隆到 `evals/` 目录中 —— 请参阅 `evals/README.md` 了解设置方法。插件基础设施测试位于 `tests/` 目录下，通过相关的 `run-*.sh` 或 `npm test` 运行。

完整的指南请参见 `skills/writing-skills/SKILL.md`。

## 更新

Superpowers 的更新因编程智能体而异，但通常是自动的。

## 许可证

MIT 许可证 —— 详情请参见 LICENSE 文件。

## 视觉伴侣遥测

由于技能和插件不会向创建者提供任何反馈，我们无法知道有多少人在使用 Superpowers。默认情况下，brainstorming 的可选视觉伴侣功能中的 Prime Radiant 标志是从我们的网站加载的。它包含了正在使用的 Superpowers 版本，但不包含有关你的项目、提示或编程智能体的任何详细信息。我们看不到你的点击或你正在构建的内容。这有助于我们大致了解有多少人正在使用 Superpowers 以及他们使用的是哪个版本。这是 100% 可选的。要禁用此功能，请将环境变量 `SUPERPOWERS_DISABLE_TELEMETRY` 设置为任何真值。Superpowers 也尊重 Claude Code 的 `DISABLE_TELEMETRY` 和 `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` 退出选项。

## 社区

Superpowers 由 [Jesse Vincent](https://blog.fsck.com) 和 [Prime Radiant](https://primeradiant.com) 的其他人员构建。

- **Discord**：[加入我们](https://discord.gg/35wsABTejz)，获取社区支持、提问和分享你使用 Superpowers 构建的内容
- **Issues**：https://github.com/obra/superpowers/issues
- **版本发布通知**：[注册](https://primeradiant.com/superpowers/) 以获取新版本的发布通知
