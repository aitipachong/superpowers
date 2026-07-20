# Superpowers 开发工作流项目深度分析

> 适用范围：面向 FDE（前端开发工程师）、软件开发人员、PM（产品经理）的 Superpowers 使用指南
> 目标：通过标准化 skill 使用路径，确保项目不偏离目标、交付质量可控

---

## 目录

1. [Superpowers 是什么](#一superpowers-是什么)
2. [核心工作流](#二核心工作流)
3. [技能体系源码剖析](#三技能体系源码剖析)
4. [核心技能详解](#四核心技能详解)
5. [进阶用法与实战技巧](#五进阶用法与实战技巧)
6. [常见误区与避坑指南](#六常见误区与避坑指南)
7. [总结](#七总结)

---

## 一、Superpowers 是什么

### 1.1 项目定位

**Superpowers** 是一套**完整的 AI 辅助软件开发方法论**，专为与编程智能体（如 Claude Code、Codex、GitHub Copilot CLI、Kimi Code 等）协作而设计。它通过一系列可组合的"技能"（skills）来规范 AI 编码行为，确保 AI 在开发过程中不会跳过关键步骤、不会随意猜测、不会盲目编码。

| 维度               | 说明                                                                                        |
| ------------------ | ------------------------------------------------------------------------------------------- |
| **性质**     | 插件化的开发方法论框架                                                                      |
| **目标**     | 让 AI 编码助手遵循成熟的软件工程流程                                                        |
| **核心哲学** | 系统化优于临时性、证据胜于声明、测试先于代码                                                |
| **适用对象** | 使用 AI 辅助编程的开发者、团队、PM                                                          |
| **支持平台** | Claude Code、Codex App/CLI、Cursor、GitHub Copilot CLI、Kimi Code、OpenCode、Pi 等 10+ 平台 |

### 1.2 它能解决什么问题

传统 AI 编码助手常见的问题：

- **直接编码，不问需求**：AI 看到需求就开始写代码，结果南辕北辙
- **跳过设计**：没有设计文档，想到哪写到哪
- **没有测试**："先写代码，测试以后补"
- **随机修复**：遇到 bug 就试来试去，不找根因
- **虚假完成**："应该可以了"就直接提交，实际上测试都没跑

Superpowers 通过**强制技能触发**机制解决这些问题：AI 在采取行动之前，必须先检查是否有适用的技能，并按照技能要求执行流程。🎯

### 1.3 项目结构概览

```
superpowers/
├── skills/                          # 核心技能目录
│   ├── using-superpowers/           # 技能系统引导（入口）
│   ├── brainstorming/               # 头脑风暴（需求阶段）
│   ├── writing-plans/               # 编写计划（计划阶段）
│   ├── using-git-worktrees/         # 工作树隔离（环境准备）
│   ├── subagent-driven-development/ # 子代理开发（核心开发）
│   ├── executing-plans/             # 执行计划（备选开发）
│   ├── test-driven-development/     # 测试驱动开发（编码阶段）
│   ├── systematic-debugging/        # 系统调试（调试阶段）
│   ├── requesting-code-review/      # 请求代码审查（质量门禁）
│   ├── receiving-code-review/       # 接收代码审查（反馈处理）
│   ├── dispatching-parallel-agents/ # 并行代理（加速调试）
│   ├── finishing-a-development-branch/ # 完成开发分支（集成阶段）
│   ├── verification-before-completion/ # 验证完成（诚实门槛）
│   └── writing-skills/              # 编写新技能（扩展）
├── docs/                            # 文档
│   ├── superpowers/
│   │   ├── specs/                   # 设计规格文档
│   │   └── plans/                   # 实施计划文档
│   └── README.*.md                  # 各平台安装指南
├── .github/                         # GitHub 模板
│   ├── PULL_REQUEST_TEMPLATE.md
│   └── ISSUE_TEMPLATE/
└── tests/                           # 插件基础设施测试
```

---

## 二、核心工作流

Superpowers 定义了一个完整的软件开发生命周期，共 **7 个核心阶段**。

### 2.1 阶段总览

```
项目启动
    ↓
[需求澄清] ────────→ brainstorming（头脑风暴）
    ↓                    ↓
[设计确认] ←────────── 用户确认设计文档
    ↓
[计划制定] ────────→ writing-plans（编写计划）
    ↓                    ↓
[环境准备] ────────→ using-git-worktrees（工作树隔离）
    ↓                    ↓
[开发实施] ────────→ subagent-driven-development（子代理开发）
    │                    │
    │                 [编码中] ───→ test-driven-development（TDD）
    │                    │
    │                 [出 bug] ───→ systematic-debugging（系统调试）
    │                    │
    │                 [每任务后] ─→ requesting-code-review（代码审查）
    │                    │
    │                 [收到反馈] ─→ receiving-code-review（接收审查）
    └──────────────────────┘
                           ↓
[测试验证] ────────→ verification-before-completion（验证完成）
    ↓
[集成完成] ────────→ finishing-a-development-branch（完成分支）
    ↓
[上线]
```

### 2.2 每个阶段的核心目标

| 阶段               | 核心技能                           | 防御目标       | 关键产出                                           |
| ------------------ | ---------------------------------- | -------------- | -------------------------------------------------- |
| **需求澄清** | `brainstorming`                  | 防止理解偏差   | 设计文档（`specs/YYYY-MM-DD-<topic>-design.md`） |
| **计划制定** | `writing-plans`                  | 防止实施混乱   | 实施计划（`plans/YYYY-MM-DD-<feature>.md`）      |
| **环境准备** | `using-git-worktrees`            | 防止环境污染   | 隔离工作空间 + 干净基线                            |
| **开发实施** | `subagent-driven-development`    | 防止质量滑坡   | 逐任务审查 + 代码提交                              |
| **测试验证** | `test-driven-development`        | 防止无测试代码 | 红-绿-重构循环                                     |
| **问题调试** | `systematic-debugging`           | 防止随机修复   | 根因分析报告 + 修复测试                            |
| **代码审查** | `requesting-code-review`         | 防止问题级联   | 审查报告（Critical/Important/Minor）               |
| **完成集成** | `finishing-a-development-branch` | 防止集成错误   | 合并/PR/保留/丢弃决策                              |

### 2.3 工作流程的强制性

Superpowers 的关键设计是：**技能自动触发，不可跳过**。每个技能的文件顶部都有 `EXTREMELY-IMPORTANT` 标记：

> "If you think there is even a 1% chance a skill might apply to what you are doing, you ABSOLUTELY MUST invoke the skill."

这意味着 AI 助手不会随意跳过任何步骤，从而保证开发流程的一致性。✨

**红队检查（Red Flags）：**

| 错误想法           | 正确认知                           |
| ------------------ | ---------------------------------- |
| "这只是简单问题"   | 问题即任务，检查技能               |
| "我需要更多上下文" | 技能检查先于澄清问题               |
| "让我先探索代码库" | 技能告诉你如何探索                 |
| "这不需要正式技能" | 如果技能存在，就使用它             |
| "我记得这个技能"   | 技能会演进，读取当前版本           |
| "这感觉更高效"     | 无纪律的行动浪费时间，技能防止这个 |

---

## 三、技能体系源码剖析

### 3.1 技能文件结构

每个技能都是独立的 Markdown 文件，遵循标准格式：

```markdown
---
name: skill-name
description: "触发条件和用途描述"
---

# 技能标题

## 核心原则
核心原则，1-2 句话。

## 使用时机
什么时候该用这个技能，什么时候不该用。

## 执行流程 (DOT 流程图)
用 GraphViz DOT 语法定义决策流程。

## 检查清单
绝对不能违反的规则。

## 常见错误
执行层面常见错误和应对方式。

## 红旗警告
常见的错误思维陷阱和对应的正确认知
```

**示例：** `skills/test-driven-development/SKILL.md`

```markdown
---
name: test-driven-development
description: Use when implementing any feature or bugfix, before writing implementation code
---

# Test-Driven Development (TDD)

## Overview
Write the test first. Watch it fail. Write minimal code to pass.

**Core principle:** If you didn't watch the test fail, you don't know if it tests the right thing.
```

### 3.2 技能触发机制

Superpowers 通过**插件机制**在会话启动时注入 `using-superpowers` 引导技能。这个引导技能的核心规则是：

**"在采取任何行动之前，先检查是否有适用的技能。"**

这个规则通过 `EXTREMELY-IMPORTANT` 和 `Red Flags` 表格来防止 AI "合理化"跳过技能：

### 3.3 设计哲学

#### 3.3.1 技能优先于直觉

`using-superpowers` 明确说明：用户指令 > 技能 > 默认行为。这意味着即使 AI 的"直觉"告诉它可以直接编码，技能规则也会覆盖这个直觉。

#### 3.3.2 硬门槛（HARD-GATE）

`brainstorming` 技能中有一个明确的硬门槛：

```markdown
<HARD-GATE>
Do NOT invoke any implementation skill, write any code, scaffold any project, or take any implementation action until you have presented a design and the user has approved it.
</HARD-GATE>
```

这是一个不可绕过的规则，确保**设计先于编码**。

#### 3.3.3 文件驱动的工作流

所有关键产出都是文件：

- 设计文档：`docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`
- 计划文档：`docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`
- 进度台账：`.superpowers/sdd/progress.md`

**文件驱动的好处：**

- **可审查**：用户可以随时查看文件内容
- **可恢复**：即使会话崩溃，文件仍然存在
- **可追踪**：git 历史记录完整的工作轨迹

#### 3.3.4 证据优先

`verification-before-completion` 技能的核心规则：

```markdown
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

任何"完成"、"修复了"、"通过了"的声称，都必须有实际运行的命令和输出作为证据。📝

### 3.4 技能分类体系

Superpowers 将技能分为四大类：

#### 测试类

- `test-driven-development`：RED-GREEN-REFACTOR 循环

#### 调试类

- `systematic-debugging`：四阶段根因分析
- `verification-before-completion`：验证完成

#### 协作类（核心工作流）

- `brainstorming`：苏格拉底式设计提炼
- `writing-plans`：详细实施计划
- `executing-plans`：批量执行
- `subagent-driven-development`：子代理开发
- `dispatching-parallel-agents`：并行代理
- `requesting-code-review`：请求审查
- `receiving-code-review`：接收审查
- `using-git-worktrees`：工作树隔离
- `finishing-a-development-branch`：完成分支

#### 元技能

- `writing-skills`：编写新技能
- `using-superpowers`：技能系统介绍

---

## 四、核心技能详解

### 4.1 `brainstorming`（头脑风暴）—— 需求阶段

**用途：** 在编写任何代码之前，通过苏格拉底式提问来澄清需求、探索方案、制定设计。

**核心流程：**

```
探索项目上下文 → 问澄清问题 → 提供2-3种方案 → 分章节呈现设计 → 用户确认 → 编写设计文档 → 自我审查 → 用户审核文档 → 触发 writing-plans
```

**关键原则：**

- **一次只问一个问题**：避免信息过载
- **每个项目都必须设计**：即使是一个简单的 todo 列表
- **先问后答**：通过问题理解用户的真实意图，而不是假设
- **YAGNI（你不会需要它）**：设计中去掉不必要的功能

**视觉伴侣（Visual Companion）：**

可选的浏览器工具，在需要展示 mockups、布局对比、架构图时提供可视化支持。注意：**不是每个问题都用视觉**，只有当"用户看了比读了更清楚"时才使用。

**产出：** 设计文档（`docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`）

**自我审查清单：**

- **Placeholder scan**：任何 "TBD"、"TODO"、不完整部分或模糊需求？修复它们。
- **Internal consistency**：各章节是否矛盾？架构是否与功能描述匹配？
- **Scope check**：是否足够聚焦，适合单个实施计划？
- **Ambiguity check**：任何需求是否可能有两种理解？如果是，选择一种并明确说明。

---

### 4.2 `writing-plans`（编写计划）—— 计划阶段

**用途：** 将已确认的设计转化为可执行的详细计划。

**核心要求：**

- **任务大小适中**：每个任务 2-5 分钟可完成一个步骤
- **无占位符**：禁止出现 "TBD"、"TODO"、"implement later"、"similar to Task N"
- **完整代码**：每个步骤必须包含实际代码或命令
- **精确文件路径**：每个任务指定创建/修改/测试的确切文件

**计划文档结构：**

```markdown
# [功能名称] 实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: superpowers:subagent-driven-development

**Goal:** 一句话描述
**Architecture:** 2-3 句话描述方法
**Tech Stack:** 关键技术/库

## Global Constraints

[项目级要求：版本、依赖限制、命名规则、平台要求]

---

### Task N: [组件名称]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

**Interfaces:**
- Consumes: [输入接口]
- Produces: [输出接口]

- [ ] **Step 1: 写失败测试**
- [ ] **Step 2: 运行测试看失败**
- [ ] **Step 3: 写最小实现**
- [ ] **Step 4: 运行测试看通过**
- [ ] **Step 5: 提交**
```

**自我审查清单：**

1. **Spec 覆盖度**：每个 spec 要求是否对应到任务？
2. **占位符扫描**：全文搜索 "TBD"、"TODO"、"implement later"
3. **类型一致性**：任务间函数名、签名、变量名是否一致？

---

### 4.3 `using-git-worktrees`（工作树隔离）—— 环境准备

**用途：** 确保工作在隔离的空间进行，保护主分支。

**核心流程：**

```
Step 0: 检测现有隔离（检查 GIT_DIR 和 GIT_COMMON）
Step 1: 创建隔离工作空间（优先原生工具，回退 git worktree）
Step 2: 项目环境设置（自动检测 npm/cargo/pip/go）
Step 3: 验证干净基线（运行测试套件）
```

**关键原则：**

- **不在 main/master 直接开发**
- **不在已有 worktree 中嵌套创建另一个**
- **基线测试不通过必须报告**

**目录优先级：**

用户指定 > 已有 `.worktrees/` > 已有 `worktrees/` > 默认 `.worktrees/`

**快速参考：**

| 情况                 | 操作                             |
| -------------------- | -------------------------------- |
| 已在 linked worktree | 跳过创建（Step 0）               |
| 在 submodule 中      | 按正常 repo 处理（Step 0 guard） |
| 有原生 worktree 工具 | 使用它（Step 1a）                |
| 无原生工具           | Git worktree 回退（Step 1b）     |
| `.worktrees/` 存在 | 使用它（验证已忽略）             |
| 目录未忽略           | 添加到 .gitignore + 提交         |
| 权限错误             | 沙箱回退，就地工作               |
| 基线测试失败         | 报告失败 + 询问                  |

---

### 4.4 `subagent-driven-development`（子代理开发）—— 核心开发

**用途：** 通过为每个任务派遣独立的子代理，实现高质量、快速迭代的开发。

**为什么用子代理：**

- **上下文隔离**：每个任务有独立的上下文，不受其他任务干扰
- **并行安全**：子代理之间不会互相干扰
- **质量门禁**：每个任务完成后有审查
- **持续执行**：不需要在每个任务之间等待用户确认

**执行流程：**

```
读取计划 → 创建任务列表 → 逐任务执行：

  1. 提取任务简报（scripts/task-brief）
  2. 派遣实现者子代理（implementer-prompt.md）
  3. 处理子代理状态：
     - DONE: 生成审查包，进入审查
     - DONE_WITH_CONCERNS: 评估担忧
     - NEEDS_CONTEXT: 补充上下文后重新派遣
     - BLOCKED: 分析原因，换模型/拆分任务/升级用户
  4. 任务审查（必须）：spec 合规性 + 代码质量
  5. 修复循环：Critical/Important 问题修复后重新审查
  6. 标记任务完成

最终全分支审查 → 使用 finishing-a-development-branch
```

**模型选择策略：**

| 任务类型                       | 推荐模型                | 原因             |
| ------------------------------ | ----------------------- | ---------------- |
| 机械实现（1-2 文件，清晰规格） | 快速/便宜模型           | 成本低，速度快   |
| 集成和判断任务（多文件协调）   | 标准模型                | 需要理解上下文   |
| 架构和设计任务                 | 最强模型                | 需要推理能力     |
| 审查任务                       | 按 diff 大小/复杂度匹配 | 平衡成本与准确性 |

**关键原则：**

- **不并行派遣多个实现者**（会冲突）
- **不跳过任务审查**
- **不在审查有未解决的 Critical/Important 问题时进入下一个任务**
- **所有产物通过文件传递**，不粘贴到上下文

**文件传递规范：**

| 产物     | 文件                     | 说明             |
| -------- | ------------------------ | ---------------- |
| 任务简报 | `task-N-brief.md`      | 实现者的需求文档 |
| 实现报告 | `task-N-report.md`     | 实现者的完成报告 |
| 审查包   | `review-N-package.md`  | diff + 提交列表  |
| 审查反馈 | `review-N-feedback.md` | 审查者的发现     |

---

### 4.5 `test-driven-development`（测试驱动开发）—— 编码阶段

**用途：** 确保所有代码都有测试，且测试先于代码。

**铁律：**

```
NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST
```

**红-绿-重构循环：**

```
RED（写失败测试） → Verify RED（看测试失败） → GREEN（最小实现） → Verify GREEN（看测试通过） → REFACTOR（清理） → Repeat
```

**为什么必须看测试失败：**

- 测试通过立即证明不了什么：可能测的是已有行为、可能测试本身有问题、可能测的是实现而不是需求
- 看测试失败证明测试确实在测某个东西

**常见借口的反驳：**

| 借口                       | 现实                                    |
| -------------------------- | --------------------------------------- |
| "太简单，不需要测试"       | 简单代码也会 break，测试只需 30 秒      |
| "我测试后写"               | 测试后立即通过的测试证明不了什么        |
| "我已经手动测试了"         | 手动测试是 ad-hoc，没有记录，不能重运行 |
| "删除 X 小时的工作是浪费"  | 沉没成本谬误。保留无测试代码是技术债    |
| "TDD 是教条，我 pragmatic" | TDD 就是 pragmatic，比生产环境调试快    |

**验证清单：**

- [ ] 每个新函数/方法都有测试
- [ ] 看过每个测试失败
- [ ] 每个测试因为预期原因失败（功能缺失，不是 typo）
- [ ] 写了最小代码通过每个测试
- [ ] 所有测试通过
- [ ] 输出干净（无错误、无警告）
- [ ] 测试使用真实代码（不 mock，除非不可避）
- [ ] 覆盖边缘情况和错误

---

### 4.6 `systematic-debugging`（系统调试）—— 调试阶段

**用途：** 用系统化方法找到根因，避免随机修复。

**铁律：**

```
NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST
```

**四阶段法：**

| 阶段                          | 关键活动                                       | 成功标准           |
| ----------------------------- | ---------------------------------------------- | ------------------ |
| **Phase 1: 根因调查**   | 读错误、复现、检查变更、收集证据、追踪数据流   | 理解 WHAT 和 WHY   |
| **Phase 2: 模式分析**   | 找工作正常的代码、对比参考实现、列出所有差异   | 识别差异           |
| **Phase 3: 假设与验证** | 提出单一假设、做最小改动验证、确认或提出新假设 | 确认或新假设       |
| **Phase 4: 实现修复**   | 创建失败测试、实施单一修复、验证               | Bug 解决，测试通过 |

**关键红线：**

- **3 次修复失败 → 质疑架构**：可能是架构问题，不是具体 bug
- **不看错误信息就修复**：错误信息往往包含解决方案
- **"先 quick fix 一下再调查"**：第一次修复就设定了模式，必须正确

**多组件系统调试：**

对于 CI → 构建 → 签名、API → 服务 → 数据库等多层系统，**在提出修复前添加诊断仪器**：

```bash
# Layer 1: Workflow
echo "=== Secrets available in workflow: ==="
echo "IDENTITY: ${IDENTITY:+SET}${IDENTITY:-UNSET}"

# Layer 2: Build script
echo "=== Env vars in build script: ==="
env | grep IDENTITY || echo "IDENTITY not in environment"

# Layer 3: Signing script
echo "=== Keychain state: ==="
security list-keychains
security find-identity -v
```

---

### 4.7 `requesting-code-review`（请求代码审查）—— 质量门禁

**用途：** 在问题级联之前捕获它们。

**触发时机：**

- **强制**：每个任务后、主要功能后、合并到 main 前
- **可选**：卡壳时、重构前、修复复杂 bug 后

**审查流程：**

```bash
# 1. 获取 git 范围
BASE_SHA=$(git rev-parse HEAD~1)
HEAD_SHA=$(git rev-parse HEAD)

# 2. 派遣代码审查子代理
# 使用 code-reviewer.md 模板
```

**反馈处理：**

- **Critical**：立即修复
- **Important**：修复后再继续
- **Minor**：记录，稍后处理

**关键原则：**

- **不因为"很简单"跳过审查**
- **不忽略 Critical 问题**
- **不用"感谢""你说得对"等表演式回应**，用技术描述替代

---

### 4.8 `finishing-a-development-branch`（完成开发分支）—— 集成阶段

**用途：** 验证测试通过后，提供结构化的集成选项。

**选项（正常 repo/命名分支）：**

1. 本地合并回 `<base-branch>`
2. 推送并创建 Pull Request
3. 保持分支现状（稍后处理）
4. 丢弃此工作

**选项（Detached HEAD）：**

1. 推送为新分支并创建 PR
2. 保持现状
3. 丢弃

**关键原则：**

- **测试通过是前提**：测试失败不呈现选项
- **丢弃需要确认**：要求输入 "discard" 确认
- **worktree 清理顺序**：先合并 → 再清理 worktree → 再删除分支

**快速参考：**

| 选项        | 合并 | 推送 | 保留 Worktree | 清理分支    |
| ----------- | ---- | ---- | ------------- | ----------- |
| 1. 本地合并 | yes  | -    | -             | yes         |
| 2. 创建 PR  | -    | yes  | yes           | -           |
| 3. 保留     | -    | -    | yes           | -           |
| 4. 丢弃     | -    | -    | -             | yes (force) |

---

### 4.9 `verification-before-completion`（验证完成）—— 诚实门槛

**用途：** 防止"我觉得可以了"的虚假完成。

**铁律：**

```
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

**门函数：**

```markdown
1. IDENTIFY: 什么命令能证明这个声称？
2. RUN: 执行完整命令（新鲜、完整）
3. READ: 阅读完整输出，检查退出码，统计失败数
4. VERIFY: 输出是否支持声称？
5. ONLY THEN: 做出声称
```

**常见失败：**

| 声称     | 需要                 | 不充分                 |
| -------- | -------------------- | ---------------------- |
| 测试通过 | 测试命令输出：0 失败 | 之前的运行、"应该通过" |
| 构建成功 | 构建命令：exit 0     | Linter 通过            |
| Bug 修复 | 原始症状的测试通过   | 代码变了，假设修复     |
| 需求满足 | 逐行检查清单         | 测试通过               |

**关键原则：**

- **不用 "should"、"probably"、"seems to"**
- **不在验证前表达满意**（"Great!"、"Done!"、"完美！"）
- **不基于"之前的运行""应该可以""我有信心"做出判断**

---

## 五、进阶用法与实战技巧

### 5.1 技巧一：利用 `dispatching-parallel-agents` 加速调试

**场景：** 6 个测试失败，分布在 3 个不同文件中，原因各不相同。

**做法：** 将独立的问题分派给独立的子代理并行处理：

```text
Agent 1 → 修复 agent-tool-abort.test.ts（3 个失败）
Agent 2 → 修复 batch-completion-behavior.test.ts（2 个失败）
Agent 3 → 修复 tool-approval-race-conditions.test.ts（1 个失败）
```

**结果：** 3 个问题同时解决，而非串行。

**关键：** 问题必须独立——如果修复一个可能修复另一个，则应先一起调查。

**验证：** 代理返回后：

1. 审查每个摘要——理解改变了什么
2. 检查冲突——代理是否编辑了相同代码？
3. 运行完整套件——验证所有修复一起工作
4. 抽查——代理可能犯系统性错误

---

### 5.2 技巧二：模型分级使用，节省成本

**策略：**

- **实现者**：机械任务用快速模型（如 Haiku），复杂任务用标准模型（如 Sonnet）
- **审查者**：按 diff 大小和复杂度选择模型
- **最终审查**：用最强模型（如 Opus）

**示例：**

```markdown
任务：添加一个独立的验证函数（1 文件，清晰规格）
→ 实现者：快速模型（成本低，速度快）
→ 审查者：标准模型（需要判断，但不需要推理）

任务：重构核心架构（多文件，需判断）
→ 实现者：标准模型
→ 审查者：最强模型（需要深度推理）
```

**为什么有效：** 快速模型在简单任务上往往和标准模型一样快，但成本只有 1/10。

**重要原则：** Turn count beats token price。墙钟和上下文成本取决于子代理花费的回合数，最便宜的模型在多步工作上通常花费 2-3× 的回合——总体成本更高。审查者和基于 prose 描述工作的实现者，使用中级模型作为底线。

---

### 5.3 技巧三：利用进度台账恢复会话

**场景：** 会话崩溃或压缩后，上下文丢失。

**做法：** 使用 `.superpowers/sdd/progress.md` 作为恢复地图：

```markdown
Task 1: complete (commits a1b2c3d..e4f5g6h, review clean)
Task 2: complete (commits i7j8k9l..m0n1o2p, review clean)
Task 3: in_progress
```

**恢复步骤：**

1. 读取 `progress.md` 和 `git log`
2. 已完成任务不再重新派遣
3. 从第一个未完成任务恢复

**关键：** `git clean -fdx` 会摧毁台账（它是 git-ignored 的 scratch）；如果发生，从 `git log` 恢复。

---

### 5.4 技巧四：利用文件传递减少上下文污染

**策略：** 所有产物通过文件传递，不粘贴到对话上下文。

**具体做法：**

| 产物     | 文件                     | 说明             |
| -------- | ------------------------ | ---------------- |
| 任务简报 | `task-N-brief.md`      | 实现者的需求文档 |
| 实现报告 | `task-N-report.md`     | 实现者的完成报告 |
| 审查包   | `review-N-package.md`  | diff + 提交列表  |
| 审查反馈 | `review-N-feedback.md` | 审查者的发现     |

**好处：** 上下文保持清晰，主代理只协调，不存储所有细节。

**为什么：** 你粘贴到派遣提示中的所有内容——以及子代理打印的所有内容——会在你的上下文中停留整个会话的剩余时间，并在每个后续回合中重新读取。通过文件传递工件可以控制上下文膨胀。

---

### 5.5 技巧五：PM 如何与 Superpowers 协作

**PM 视角下的 Superpowers 使用：**

1. **需求阶段**：用 `brainstorming` 确保 AI 真正理解需求，而不是假设

   - 要求 AI 提供 2-3 种方案，对比优劣
   - 确认设计文档后再进入开发
2. **计划阶段**：审查 `writing-plans` 产出的计划文档

   - 检查是否有遗漏的需求
   - 确认任务粒度适中
3. **开发阶段**：不需要 micromanage，Superpowers 自动执行

   - 每个任务后有自动审查
   - 遇到 block 时 AI 会主动询问
4. **验收阶段**：要求 AI 提供 `verification-before-completion` 证据

   - 不是"测试应该通过了"，而是"这是测试运行的输出"

**PM 应该关注的红线：**

- AI 说"我觉得可以了"但没有测试输出 → 要求运行测试
- AI 跳过设计直接编码 → 要求回到 brainstorming
- AI 说"这个 bug 很明显，直接改" → 要求走 systematic-debugging

---

### 5.6 技巧六：FDE（前端开发工程师）的 Superpowers 最佳实践

**前端开发场景：**

1. **UI 组件开发**：

   - `brainstorming` 阶段：确认组件 API、props、状态管理
   - `writing-plans` 阶段：按组件 → 样式 → 测试 → 故事书 分解任务
   - `test-driven-development`：先写测试（渲染、交互、边缘情况）
2. **与视觉伴侣配合**：

   - 当需要讨论布局、颜色、对比时，启用视觉伴侣
   - 概念性问题（如"什么是 personality"）用文本，视觉问题（如"哪个布局更好"）用浏览器
3. **并行调试**：

   - 当多个测试文件失败时，用 `dispatching-parallel-agents` 并行修复

---

### 5.7 技巧七：编写自定义技能（团队扩展）

**场景：** 团队有特定的编码规范、流程要求。

**使用 `writing-skills` 技能：**

1. **复制 `writing-skills/SKILL.md` 的模板**
2. **填写标准格式：**
   - `name`: 技能名称
   - `description`: 触发条件
   - 内容：核心原则、执行流程、检查清单
3. **保存到 `skills/<skill-name>/SKILL.md`**
4. **测试技能：** 创建测试场景，验证技能是否正确触发

**示例：自定义"前端组件开发"技能**

```markdown
---
name: frontend-component-dev
description: Use when creating or modifying React/Vue components
---

# Frontend Component Development

## 核心原则
- 每个组件一个文件
- Props 必须定义类型
- 必须包含 storybook 故事

## 执行流程
1. 检查是否有现有组件模式
2. 定义 Props 接口
3. 编写组件（使用测试驱动）
4. 添加样式
5. 添加故事

## 检查清单
- [ ] Props 类型定义完整
- [ ] 包含默认 props
- [ ] 有对应的测试文件
- [ ] 有 storybook 故事
```

---

## 六、常见误区与避坑指南

### 误区一："需求很简单，不需要 brainstorm"

**风险：** 未探索的假设导致方向偏差，后期返工成本高。
**正确做法：** 无论多简单，都使用 brainstorming。简单项目的设计可以只有几句话，但必须有。

### 误区二："计划里有 TODO 没关系，以后补"

**风险：** 计划不完整导致执行时遗漏，工程师无所适从。
**正确做法：** writing-plans 的"无占位符"规则是硬性的。任何 TBD/TODO 都必须在计划阶段解决。

### 误区三："这次改动小，直接改 main 就行"

**风险：** 污染主分支，影响其他开发者，回滚困难。
**正确做法：** 无论改动大小，使用 using-git-worktrees 创建隔离空间。

### 误区四："先写代码，测试稍后补"

**风险：** 测试可能测的是已实现行为而非需求行为，测试质量低，且无法验证测试是否有效。
**正确做法：** 严格执行 test-driven-development。如果已经先写了代码，删除它，重新开始。

### 误区五："这个 bug 很明显，直接改就行"

**风险：** 症状修复掩盖根因，可能引入新 bug，同一问题反复出现。
**正确做法：** 使用 systematic-debugging，完成四阶段调查后再修复。如果 3 次修复失败，质疑架构。

### 误区六："审查就是走个形式，肯定没问题"

**风险：** 问题在任务间级联，到后期修复成本指数级增长。
**正确做法：** 每个任务后都使用 requesting-code-review。审查是质量门禁，不是形式主义。

### 误区七："测试应该通过了，我就不运行了"

**风险：** 未验证的代码可能包含未发现的错误，破坏信任。
**正确做法：** 使用 verification-before-completion。任何完成声称前，必须有运行的命令和输出作为证据。

---

## 七、总结

Superpowers 的 skill 体系是一个**防御性编程方法论在 AI 辅助开发中的体现**。每个 skill 都是一道质量闸门：

| 阶段 | Skill                             | 防御目标     |
| ---- | --------------------------------- | ------------ |
| 需求 | brainstorming                     | 防止理解偏差 |
| 计划 | writing-plans                     | 防止实施混乱 |
| 环境 | using-git-worktrees               | 防止环境污染 |
| 开发 | subagent-driven-development + TDD | 防止质量滑坡 |
| 调试 | systematic-debugging              | 防止随机修复 |
| 审查 | requesting/receiving-code-review  | 防止问题级联 |
| 完成 | verification-before-completion    | 防止虚假完成 |
| 集成 | finishing-a-development-branch    | 防止集成错误 |

**核心信条：** 不跳过一个 skill，不跳过一步验证。证据先于声称，测试先于代码，审查先于合并。这样，你的项目从需求到上线，每一步都有据可查、有质可保。🎯

对于 FDE、开发人员和 PM 来说，Superpowers 的价值在于：**它把成熟的软件工程实践编码到 AI 的行为中，确保 AI 助手不会成为"快速但错误"的编码机器。** 通过遵循这些流程，团队可以充分利用 AI 的效率，同时保持代码质量和可维护性。

---

**参考文档：**

- [Superpowers GitHub 仓库](https://github.com/obra/superpowers)
- [官方文档](https://primeradiant.com/superpowers/)
- [Discord 社区](https://discord.gg/35wsABTejz)
- [Release Notes](RELEASE-NOTES.md)
- [贡献指南](CODE_OF_CONDUCT.md)

**安装 Superpowers：**

- Claude Code: `/plugin install superpowers@claude-plugins-official`
- Codex CLI: `/plugins` → 搜索 superpowers
- GitHub Copilot CLI: `copilot plugin install superpowers@superpowers-marketplace`
- Kimi Code: `/plugins` → Marketplace → Superpowers
- 其他平台：参见 README.md 各平台安装说明

---

*本文档基于 Superpowers 项目源码分析生成，涵盖 skills/ 目录下所有核心技能文件及项目文档。*
*生成日期：2026-07-17*
