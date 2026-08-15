---
name: traject
description: 项目索引与上下文导航 — 解决大项目 AI 会话的上下文断裂问题。通过结构化目录索引让 AI 按词典方式定位代码，而非通读项目。
disable-model-invocation: true
---

# Traject — 项目索引与上下文导航

## 命令分派

当用户输入以下命令时，按对应协议执行。所有命令的详细步骤、完成标准和边界情况见 [`references/protocol.md`](references/protocol.md)。

| 用户输入 | 执行动作 | 协议章节 |
|---------|---------|---------|
| `Traject 初始化` | 首次生成项目索引（三阶段：规划→分批生成→收尾） | protocol.md 第一章 |
| `Traject 加载` | 加载项目概览到当前会话 | protocol.md 第二章 |
| `Traject 更新` | 增量更新索引（检测 git 变更） | protocol.md 第三章 |
| `Traject 继续` | 恢复中断的初始化任务 | protocol.md 第四章 |

### 分派规则

1. 提取用户输入，匹配前缀（带空格，避免中文分词歧义）
2. 匹配 `Traject 初始化` → 执行初始化协议
3. 匹配 `Traject 加载` → 执行加载协议
4. 匹配 `Traject 更新` → 执行更新协议
5. 匹配 `Traject 继续` → 执行恢复协议
6. 无匹配时，用户可能是在日常提问 → 按「查询定位协议」处理（见 AGENTS.md）

## 加载协议（新会话启动）

当用户输入 `Traject 加载` 或在新会话中首次交互时，执行以下步骤：

1. **检测**：检查当前项目根目录是否存在 `.traject/manifest.md`
2. **加载**（若存在）：
   - 读取 `.traject/manifest.md`
   - 读取 `.traject/exclude.yaml`
   - 向用户报告项目一级目录结构、索引版本、索引目录总数
   - 等待用户提出具体问题后再深入
3. **提示**（若不存在）：
   - 提示用户：「未检测到 Traject 索引。输入 "Traject 初始化" 开始建立索引。」

## 查询定位协议（日常使用）

当用户提出具体问题时（如「XX 功能在哪里？」），而非输入 Traject 命令时：

1. 首先确认 `.traject/manifest.md` 存在（若不存在，提示初始化）
2. 从 `.traject/manifest.md` 定位到相关的一级目录
3. 读取 `.traject/{一级目录路径}/.traject.md`，定位到二级目录
4. 逐级深入，每次读取 `.traject/{当前路径}/.traject.md`，直到定位到具体文件
5. **只读取目标文件的完整代码**，回答用户问题

**禁止行为**：
- ❌ 在未读取 manifest 的情况下直接扫描项目目录
- ❌ 一次性读取所有 `.traject/**/.traject.md` 文件
- ❌ 跳过索引直接读取代码文件

## 核心约束（所有命令通用）

执行任何命令时，必须遵守以下约束：

### 格式约束
- 每个 `.traject.md` 不超过 35 行
- 每个文件职责描述不超过 10 个字
- 摘要软约束 15-25 字，超过 30 字时自我审查是否冗余
- 每个文件最多列出 3 个关键符号
- 只描述**直接子级**，禁止跨级

### 生成约束
- 自下而上：叶子节点先于父节点生成
- 叶子节点读源码，父节点读子级 `.traject.md`
- 禁止跨级汇总（父节点不得读取孙子节点的 `.traject.md`）
- 排除配置分层：通用 Profile + 平台 Profile → 合并写入项目 `.traject/exclude.yaml`
- AI 启发式判断：初始化时自动检测可疑目录/文件，按置信度分级建议排除

### 批量约束
- 每批处理 5-10 个目录
- 每批完成后更新 `traversal-state.json`
- 每批完成后询问用户是否继续

### 排除约束
- 通用排除项定义在 `references/excludes/common.yaml`（所有项目加载）
- 平台排除项定义在 `references/excludes/` 下的对应文件（按项目类型自动加载）
- 项目类型检测依据：`pubspec.yaml`（Flutter）、`package.json`（Node.js）等标志文件
- 多 Profile 可合并：同时检测到多个平台标志时，合并所有对应规则
- 路径级排除：通过 `exclude_paths` 字段匹配特定路径（如 `example/macos/Runner/`）
- AI 判断规则定义在 `references/heuristics.md`，仅在 Profile 规则未覆盖的范围内生效

## 参考文件

详细规范见以下文件，按需加载：

| 文件 | 内容 | 何时加载 |
|------|------|---------|
| [`references/conventions.md`](references/conventions.md) | 格式硬性约束、拓扑排序、描述规则 | 生成 `.traject.md` 时 |
| [`references/exclude.yaml`](references/exclude.yaml) | 排除配置模板（合并后写入项目） | 初始化和更新时 |
| [`references/excludes/`](references/excludes/) | 分层排除 Profile（common.yaml、flutter.yaml 等） | 初始化时自动检测项目类型并合并 |
| [`references/heuristics.md`](references/heuristics.md) | AI 启发式判断规则（硬规则 + 软规则） | 初始化扫描阶段 |
| [`references/templates.md`](references/templates.md) | 所有文件模板 | 生成任何文件时 |
| [`references/protocol.md`](references/protocol.md) | 完整命令步骤 + 完成标准 | 执行任何命令时 |

## 关键文件路径

| 路径 | 说明 |
|------|------|
| `~/.claude/skills/traject/SKILL.md` | 本文件（Skill 入口） |
| `~/.claude/skills/traject/references/protocol.md` | 命令协议（最重要） |
| `~/.claude/skills/traject/references/templates.md` | 文件模板 |
| `~/.claude/skills/traject/references/conventions.md` | 格式规范 |
| `~/.claude/skills/traject/references/exclude.yaml` | 排除配置模板 |
| `~/.claude/skills/traject/references/excludes/common.yaml` | 通用排除 Profile |
| `~/.claude/skills/traject/references/excludes/flutter.yaml` | Flutter 平台排除 Profile |
| `~/.claude/skills/traject/references/heuristics.md` | AI 启发式判断规则 |

## 项目生成文件

在目标项目中，Skill 会生成以下文件：

| 路径 | 说明 |
|------|------|
| `.traject/manifest.md` | 根索引入口 |
| `.traject/exclude.yaml` | 项目排除配置 |
| `.traject/plan/traversal-state.json` | 遍历进度状态 |
| `.traject/**/.traject.md` | 每个目录的索引文件（镜像项目目录结构存储于 `.traject/` 下） |
| `AGENTS.md` | AI 协议（Cursor/Windsurf 自动加载） |
| `README.md` | 开发者使用说明（新建或追加） |

## 快速参考

```
Traject 初始化  →  首次建立索引（分步执行，可中断恢复）
Traject 加载    →  新会话加载项目概览
Traject 更新    →  代码变更后增量更新索引
Traject 继续    →  恢复中断的初始化任务
直接提问        →  AI 按索引自动定位代码
```