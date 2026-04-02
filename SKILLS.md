# Skills 索引

> PipeDog-SkillHub 当前 Skills 总览，按能力域整理，便于快速查找、安装和复用。

## 统计

- Skill 总数：**14**
- 主目录：[`skills/`](./skills/)
- 安装工具：[`skill-manager`](./skill-manager)

## 目录

- [架构与运行时](#架构与运行时)
- [Prompt 与上下文](#prompt-与上下文)
- [治理、权限与验证](#治理权限与验证)
- [Skill 工具与包装](#skill-工具与包装)
- [按名称速查](#按名称速查)

---

## 架构与运行时

聚焦 Agent / Tool / MCP 运行时架构设计、执行链路和工程实践。

| Skill | 说明 | 路径 |
|---|---|---|
| [agent-lifecycle-management](./skills/agent-lifecycle-management/) | 管理 Agent 生命周期：创建、上下文继承、权限塑形、转录记录、流式进度与清理。 | `skills/agent-lifecycle-management/` |
| [flutter-architecture](./skills/flutter-architecture/) | Flutter 四层组件化 + MVVM 架构规范，适合新项目搭建、模块创建和架构评审。 | `skills/flutter-architecture/` |
| [hook-governance-layer](./skills/hook-governance-layer/) | 设计或审查基于 Hook 的治理层，用于工具执行前后拦截、权限介入和上下文注入。 | `skills/hook-governance-layer/` |
| [mcp-integration-plane](./skills/mcp-integration-plane/) | 设计或调试 MCP 集成层，覆盖连接、鉴权、资源发现、超时、恢复和输出持久化。 | `skills/mcp-integration-plane/` |
| [multi-agent-orchestration](./skills/multi-agent-orchestration/) | 设计或运行多 Agent 协作系统，明确角色、权限、上下文、MCP 和清理策略。 | `skills/multi-agent-orchestration/` |
| [tool-runtime-pipeline](./skills/tool-runtime-pipeline/) | 设计或审查工具执行流水线，覆盖校验、Hook、权限、执行、结果映射与异常处理。 | `skills/tool-runtime-pipeline/` |

---

## Prompt 与上下文

聚焦系统提示词装配、上下文压缩、缓存效率和行为规范。

| Skill | 说明 | 路径 |
|---|---|---|
| [behavior-institutionalization](./skills/behavior-institutionalization/) | 将期望行为沉淀为可执行的提示词制度，适合系统提示词、行为准则和协作规范设计。 | `skills/behavior-institutionalization/` |
| [context-hygiene-system](./skills/context-hygiene-system/) | 设计长上下文卫生系统，覆盖压缩、裁剪、恢复、附件与队列治理。 | `skills/context-hygiene-system/` |
| [prompt-assembly-architecture](./skills/prompt-assembly-architecture/) | 设计 Prompt 装配架构，明确静态前缀、动态边界、Section Registry 与缓存友好策略。 | `skills/prompt-assembly-architecture/` |
| [prompt-cache-economics](./skills/prompt-cache-economics/) | 优化 Prompt Cache 的命中率与成本，适用于前缀稳定性、分叉复用和指令预算设计。 | `skills/prompt-cache-economics/` |

---

## 治理、权限与验证

聚焦安全边界、权限决策、风险控制与独立验证。

| Skill | 说明 | 路径 |
|---|---|---|
| [blast-radius-permission](./skills/blast-radius-permission/) | 设计或审查权限系统，兼顾 allow / ask / deny、自动模式和高风险操作确认。 | `skills/blast-radius-permission/` |
| [verification-agent](./skills/verification-agent/) | 设计或运行独立验证 Agent，通过命令证据给出 PASS / FAIL / PARTIAL 结论。 | `skills/verification-agent/` |

---

## Skill 工具与包装

聚焦 Skill 的安装、分发、发现和工作流封装。

| Skill | 说明 | 路径 |
|---|---|---|
| [skill-manager](./skills/skill-manager/) | Skill 安装工具，支持 `list`、`search`、`install`、`install-global` 等命令。 | `skills/skill-manager/` |
| [skill-workflow-packaging](./skills/skill-workflow-packaging/) | 将工作流封装为可发现、可触发、可预算的 Skill，适合 Skill 系统和命令化能力设计。 | `skills/skill-workflow-packaging/` |

---

## 按名称速查

| 名称 | 分类 |
|---|---|
| `agent-lifecycle-management` | 架构与运行时 |
| `behavior-institutionalization` | Prompt 与上下文 |
| `blast-radius-permission` | 治理、权限与验证 |
| `context-hygiene-system` | Prompt 与上下文 |
| `flutter-architecture` | 架构与运行时 |
| `hook-governance-layer` | 架构与运行时 |
| `mcp-integration-plane` | 架构与运行时 |
| `multi-agent-orchestration` | 架构与运行时 |
| `prompt-assembly-architecture` | Prompt 与上下文 |
| `prompt-cache-economics` | Prompt 与上下文 |
| `skill-manager` | Skill 工具与包装 |
| `skill-workflow-packaging` | Skill 工具与包装 |
| `tool-runtime-pipeline` | 架构与运行时 |
| `verification-agent` | 治理、权限与验证 |

---

## 安装示例

```bash
# 列出所有 Skills
skill-manager list

# 搜索 Skills
skill-manager search prompt

# 安装到当前项目
skill-manager install flutter-architecture

# 安装到 Codex
skill-manager install flutter-architecture codex

# 全局安装
skill-manager install-global flutter-architecture

# 全局安装到 Codex
skill-manager install-global flutter-architecture codex
```
