# Visual Companion Guide

基于浏览器的视觉头脑风暴辅助工具，用于展示原型、图表和选项。

## 何时使用

按问题判断，而非按会话判断。测试标准：**用户通过视觉方式理解比阅读更好吗？**

**当内容本身是视觉化的时使用浏览器：**

- **UI 原型** — 线框图、布局、导航结构、组件设计
- **架构图** — 系统组件、数据流、关系映射
- **并排视觉对比** — 对比两种布局、两种配色方案、两种设计方向
- **设计润色** — 当问题是关于外观和感觉、间距、视觉层次时
- **空间关系** — 状态机、流程图、实体关系图

**当内容是文本或表格时使用终端：**

- **需求和范围问题** — "X 是什么意思？"、"哪些功能在范围内？"
- **概念性的 A/B/C 选择** — 从文字描述的方法中挑选
- **权衡列表** — 利弊、对比表
- **技术决策** — API 设计、数据建模、架构方法选择
- **澄清问题** — 任何答案都是文字而非视觉偏好的问题

关于 UI 主题的问题不自动成为视觉问题。"你想要什么样的向导？"是概念性的——用终端。"这些向导布局哪个感觉更好？"是视觉的——用浏览器。

## 工作原理

服务器监听一个目录中的 HTML 文件，并将最新的文件提供给浏览器。你将 HTML 内容写入 `screen_dir`，用户在其浏览器中查看并点击选择选项。选择结果会记录到 `state_dir/events` 中，供你在下一轮读取。

**内容片段 vs 完整文档：** 如果你的 HTML 文件以 `<!DOCTYPE` 或 `<html` 开头，服务器会原样提供（仅注入辅助脚本）。否则，服务器会自动将你的内容包裹在框架模板中——添加标题、CSS 主题、连接状态以及所有交互基础设施。**默认写内容片段。** 只有当你需要完全控制页面时才写完整文档。

## 启动会话

```bash
# 在用户批准辅助工具之后再启动。--open 会自动在浏览器打开
# 第一个屏幕；--project-dir 会持久化原型并支持同端口重启。
scripts/start-server.sh --project-dir /path/to/project --open

# 返回: {"type":"server-started","port":52341,
#           "url":"http://localhost:52341/?key=ab12…",
#           "screen_dir":"/path/to/project/.superpowers/brainstorm/12345-1706000000/content",
#           "state_dir":"/path/to/project/.superpowers/brainstorm/12345-1706000000/state"}
```

从响应中保存 `screen_dir` 和 `state_dir`。使用 `--open` 时，浏览器会在你推送第一个屏幕时自动打开——你不需要要求用户打开，但仍应分享 URL 作为备用（无头/远程设置不会自动打开）。

**URL 包含会话密钥（`?key=…`）。** 服务器会拒绝任何不带密钥的请求，所以始终将 **`url` 字段的完整 URL** 提供给用户——不要去掉查询字符串，也不要只给出裸 `http://host:port`。密钥控制 HTTP 和 WebSocket 访问权限，因此流浪的浏览器标签或网络中的其他机器无法读取屏幕内容或注入事件。首次加载后，浏览器通过 cookie 记住密钥，因此刷新和 `/files/*` 资源无需重复传递密钥。

**查找连接信息：** 服务器将其启动 JSON 写入 `$STATE_DIR/server-info`。如果你在后台启动了服务器且没有捕获标准输出，请读取该文件以获取 URL 和端口。使用 `--project-dir` 时，检查 `<project>/.superpowers/brainstorm/` 获取会话目录。

**注意：** 将项目根目录作为 `--project-dir` 传递，以便原型持久化在 `.superpowers/brainstorm/` 中，并在服务器重启后存活。如果不设置，文件会进入 `/tmp` 并被清理。提醒用户将 `.superpowers/` 添加到 `.gitignore`（如果尚未添加）。

**按平台启动服务器：**

**Claude Code:**

```bash
# 默认模式有效 — 脚本本身会将服务器放在后台运行。
scripts/start-server.sh --project-dir /path/to/project --open
```

在 Windows 上，脚本会自动检测并切换到前台模式（这会阻塞工具调用）。在 Bash 工具调用上使用 `run_in_background: true`，让服务器在对话轮次之间存活，然后在下一轮读取 `$STATE_DIR/server-info` 获取 URL 和端口。

**Codex:**

```bash
# Codex 会回收后台进程。脚本会自动检测 CODEX_CI 并
# 切换到前台模式。正常运行即可 — 无需额外参数。
scripts/start-server.sh --project-dir /path/to/project --open
```

**Copilot CLI:**

```bash
# 使用 --foreground 并通过 bash 工具以 mode: "async" 启动服务器
# 让进程在轮次之间存活。如需后续交互，记录返回的 shellId 用于
# read_bash / stop_bash。
scripts/start-server.sh --project-dir /path/to/project --open --foreground
```

**其他环境：** 服务器必须在对话轮次之间保持后台运行。如果你的环境会回收分离的进程，请使用 `--foreground` 并通过你的平台的后台执行机制启动命令。

如果 URL 从你的浏览器无法访问（常见于远程/容器化设置），请绑定非回环主机：

```bash
scripts/start-server.sh \
  --project-dir /path/to/project \
  --host 0.0.0.0 \
  --url-host localhost
```

使用 `--url-host` 控制返回 URL JSON 中打印的主机名。

## 循环流程

1. **检查服务器是否存活**，然后 **将 HTML 写入** `screen_dir` 中的新文件：

   - **必需：在引用 URL 或推送屏幕之前确认服务器存活。** 检查 `$STATE_DIR/server-info` 是否存在且 `$STATE_DIR/server-stopped` 是否不存在。如果服务器已关闭，使用 **相同的 `--project-dir`** 重启 `start-server.sh` —— 它会复用同一个端口，因此用户打开的标签会自动重新连接（服务器关闭时显示"暂停"覆盖层），你无需发送新的 URL。服务器在空闲 4 小时后自动退出（可用 `--idle-timeout-minutes` 配置）。
   - 使用语义化文件名：`platform.html`、`visual-style.html`、`layout.html`
   - **永不复用文件名** —— 每个屏幕都使用新文件
   - 使用你的文件创建工具 —— **不要使用 cat/heredoc**（会在终端中输出噪音）
   - 服务器自动提供最新的文件
2. **告诉用户会发生什么，并结束你的轮次：**

   - 每步提醒 URL（不仅限于第一步）
   - 简要文字概述屏幕上显示的内容（例如，"展示首页的 3 种布局选项"）
   - 请用户在终端回复："看一看，告诉我你的看法。如果愿意，可以点击选择一个选项。"
3. **在你的下一轮** —— 用户在终端回复后：

   - 读取 `$STATE_DIR/events`（如果存在）—— 包含用户浏览器交互（点击、选择）的 JSON 行数据
   - 与用户的终端文本合并以获得完整信息
   - 终端消息是主要反馈；`state_dir/events` 提供结构化的交互数据
4. **迭代或推进** —— 如果反馈改变了当前屏幕，写一个新文件（例如，`layout-v2.html`）。只有当前步骤已验证时才推进到下一个问题。
5. **返回终端时卸载** —— 当下一步不需要浏览器时（例如，澄清问题、权衡讨论），推送一个等待屏幕以清除过时内容：

   ```html
   <!-- 文件名: waiting.html (或 waiting-2.html 等) -->
   <div style="display:flex;align-items:center;justify-content:center;min-height:60vh">
     <p class="subtitle">Continuing in terminal...</p>
   </div>
   ```

   这防止用户在对话已经继续时盯着已解决的选项。当下一个视觉问题出现时，像往常一样推送新的内容文件。
6. 重复直到完成。

## 编写内容片段

只写放入页面的内容。服务器会自动将其包裹在框架模板中（标题、主题 CSS、连接状态以及所有交互基础设施）。

**最小示例：**

```html
<h2>Which layout works better?</h2>
<p class="subtitle">Consider readability and visual hierarchy</p>

<div class="options">
  <div class="option" data-choice="a" onclick="toggleSelect(this)">
    <div class="letter">A</div>
    <div class="content">
      <h3>Single Column</h3>
      <p>Clean, focused reading experience</p>
    </div>
  </div>
  <div class="option" data-choice="b" onclick="toggleSelect(this)">
    <div class="letter">B</div>
    <div class="content">
      <h3>Two Column</h3>
      <p>Sidebar navigation with main content</p>
    </div>
  </div>
</div>
```

就这样。不需要 `<html>`、不需要 CSS、不需要 `<script>` 标签。服务器提供了所有这些。

## 可用的 CSS 类

框架模板为你的内容提供以下 CSS 类：

### 选项（A/B/C 选择）

```html
<div class="options">
  <div class="option" data-choice="a" onclick="toggleSelect(this)">
    <div class="letter">A</div>
    <div class="content">
      <h3>Title</h3>
      <p>Description</p>
    </div>
  </div>
</div>
```

**多选：** 在容器上添加 `data-multiselect` 以允许用户选择多个选项。每次点击会切换该项的选中样式。

```html
<div class="options" data-multiselect>
  <!-- 同样的选项标记 —— 用户可以选中/取消选中多个 -->
</div>
```

### 卡片（视觉设计）

```html
<div class="cards">
  <div class="card" data-choice="design1" onclick="toggleSelect(this)">
    <div class="card-image"><!-- mockup content --></div>
    <div class="card-body">
      <h3>Name</h3>
      <p>Description</p>
    </div>
  </div>
</div>
```

### 原型容器

```html
<div class="mockup">
  <div class="mockup-header">Preview: Dashboard Layout</div>
  <div class="mockup-body"><!-- your mockup HTML --></div>
</div>
```

### 分屏视图（并排）

```html
<div class="split">
  <div class="mockup"><!-- left --></div>
  <div class="mockup"><!-- right --></div>
</div>
```

### 优缺点

```html
<div class="pros-cons">
  <div class="pros"><h4>Pros</h4><ul><li>Benefit</li></ul></div>
  <div class="cons"><h4>Cons</h4><ul><li>Drawback</li></ul></div>
</div>
```

### 原型元素（线框构建块）

```html
<div class="mock-nav">Logo | Home | About | Contact</div>
<div style="display: flex;">
  <div class="mock-sidebar">Navigation</div>
  <div class="mock-content">Main content area</div>
</div>
<button class="mock-button">Action Button</button>
<input class="mock-input" placeholder="Input field">
<div class="placeholder">Placeholder area</div>
```

### 排版和分区

- `h2` — 页面标题
- `h3` — 分区标题
- `.subtitle` — 标题下方的次要文本
- `.section` — 带底部边距的内容块
- `.label` — 小型大写标签文本

## 浏览器事件格式

当用户点击浏览器中的选项时，他们的交互会记录到 `$STATE_DIR/events`（每行一个 JSON 对象）。推送新屏幕时，文件会自动清空。

```jsonl
{"type":"click","choice":"a","text":"Option A - Simple Layout","timestamp":1706000101}
{"type":"click","choice":"c","text":"Option C - Complex Grid","timestamp":1706000108}
{"type":"click","choice":"b","text":"Option B - Hybrid","timestamp":1706000115}
```

完整的事件流展示了用户的探索路径 —— 他们可能在确定选择前点击多个选项。最后一个 `choice` 事件通常是最终选择，但点击模式可能揭示犹豫或值得询问的偏好。

如果 `$STATE_DIR/events` 不存在，说明用户未与浏览器交互——仅使用他们的终端文本。

## 设计技巧

- **按问题缩放保真度** — 布局用线框图，润色问题用精细设计
- **在每个页面上解释问题** — "哪个布局感觉更专业？" 而不是仅写 "选一个"
- **推进前迭代** — 如果反馈改变了当前屏幕，写一个新版本
- **每个屏幕最多 2-4 个选项**
- **在重要时展示真实内容** — 对于摄影作品集，使用真实图片（Unsplash）。占位内容会掩盖设计问题。
- **保持原型简单** — 聚焦布局和结构，而非像素级完美设计

## 文件命名

- 使用语义化名称：`platform.html`、`visual-style.html`、`layout.html`
- 永不复用文件名 —— 每个屏幕必须是新文件
- 迭代时：添加版本后缀如 `layout-v2.html`、`layout-v3.html`
- 服务器按修改时间提供最新的文件

## 清理

```bash
scripts/stop-server.sh $SESSION_DIR
```

如果会话使用了 `--project-dir`，原型文件会持久化在 `.superpowers/brainstorm/` 中供日后参考。只有 `/tmp` 会话会在停止时被删除。

## 参考

- 框架模板（CSS 参考）：`scripts/frame-template.html`
- 辅助脚本（客户端）：`scripts/helper.js`
