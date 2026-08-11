# Traject 模板文件

本文档包含所有生成文件的模板。在实现时，方括号 `[...]` 中的内容需替换为实际值。

---

## 一、.traject.md 模板（每个目录下的索引文件）

```markdown
# 目录索引: [相对路径]

## 职责
[一句话职责，不超过20字]

## 子目录
| 目录名 | 职责 |
|--------|------|
| [subdir]/ | [职责描述] |

## 文件
| 文件名 | 职责 | 关键符号 |
|--------|------|---------|
| [filename] | [≤10字] | [sym1], [sym2] |

## 依赖
[依赖的其他目录，无依赖则省略此节]
```

### 示例：src/components/

```markdown
# 目录索引: src/components/

## 职责
可复用 UI 组件

## 子目录
| 目录名 | 职责 |
|--------|------|
| buttons/ | 按钮组件集合 |
| forms/ | 表单组件集合 |
| layouts/ | 布局组件集合 |

## 文件
| 文件名 | 职责 | 关键符号 |
|--------|------|---------|
| index.ts | 组件统一导出 | export * |
| theme.ts | 主题配置 | ThemeConfig, defaultTheme |

## 依赖
src/utils/, src/hooks/
```

### 示例：src/components/buttons/（叶子目录）

```markdown
# 目录索引: src/components/buttons/

## 职责
按钮组件集合

## 文件
| 文件名 | 职责 | 关键符号 |
|--------|------|---------|
| PrimaryButton.tsx | 主要按钮 | PrimaryButton |
| SecondaryButton.tsx | 次要按钮 | SecondaryButton |
| IconButton.tsx | 图标按钮 | IconButton |
| index.ts | 统一导出 | export * |
```

---

## 二、manifest.md 模板（根索引）

```markdown
---
branch: [当前分支名]
generated_at: [生成时间 ISO 8601]
total_dirs: [已索引目录总数]
total_files: [已索引文件总数]
---

# 项目索引

## 项目概述
[项目名称或一句话描述]

## 一级目录
| 目录名 | 职责 |
|--------|------|
| [dir1]/ | [职责描述] |
| [dir2]/ | [职责描述] |

## 排除的目录
以下目录不参与索引：
[node_modules/, vendor/, .git/, ...]

## 索引统计
- 索引目录数：[N]
- 索引文件数：[M]
- 生成时间：[时间]
- 索引版本：1.0
```

---

## 三、traversal-state.json 模板（进度状态）

```json
{
  "version": "1.0",
  "created_at": "",
  "last_updated": "",
  "total_dirs": 0,
  "processed_dirs": 0,
  "pending_dirs": [],
  "failed_dirs": [],
  "current_batch": 0,
  "batch_size": 7,
  "excluded_dirs": [],
  "status": "not_started",
  "status_note": "等待用户确认排除列表"
}
```

### 状态流转

```
not_started → plan_ready → generating → generation_complete → completed
                 ↑                          │
                 │                          ↓
                 └────── 恢复续跑 ←──────── failed（部分失败）
```

### 字段说明

| 字段 | 类型 | 说明 |
|------|------|------|
| `version` | string | 状态格式版本 |
| `created_at` | string | 首次创建时间 |
| `last_updated` | string | 最后更新时间 |
| `total_dirs` | number | 待处理目录总数 |
| `processed_dirs` | number | 已处理目录数 |
| `pending_dirs` | array | 待处理目录列表（按拓扑序排列） |
| `failed_dirs` | array | 处理失败的目录 |
| `current_batch` | number | 当前批次号 |
| `batch_size` | number | 每批处理目录数 |
| `excluded_dirs` | array | 已排除的目录列表 |
| `status` | string | 当前状态 |
| `status_note` | string | 状态备注 |

---

## 四、AGENTS.md 模板（AI 协议文件）

```markdown
# Traject 项目索引协议

本项目使用 Traject 索引系统，所有 AI 助手必须严格遵守以下协议。

## 核心原则

AI **永远不应该**在没有先读取索引的情况下直接扫描项目目录。
查询路径必须遵循：根索引 → 一级目录索引 → 二级目录索引 → ... → 目标文件。

## 一、新会话启动协议

当新会话开始，AI 必须：

1. **检测**：检查项目根目录是否存在 `.traject/manifest.md`
2. **加载**（若存在）：
   - 读取 `.traject/manifest.md`
   - 读取 `.traject/exclude.yaml`
   - 向用户报告：项目一级目录结构、索引版本、索引目录总数
   - 等待用户提出具体问题后再深入
3. **提示**（若不存在）：
   - 提示用户：「未检测到 Traject 索引。输入 "Traject 初始化" 开始建立索引。」

## 二、查询定位协议

当用户提出具体问题（如"XX 功能在哪里？"）时，AI 必须：

1. 从 `.traject/manifest.md` 定位到相关的一级目录
2. 读取对应一级目录的 `.traject.md`，定位到二级目录
3. 逐级深入，直到定位到具体文件
4. **只读取目标文件的完整代码**，回答用户问题

**禁止行为**：
- ❌ 在未读取 manifest 的情况下直接扫描项目
- ❌ 一次性读取所有 `.traject.md` 文件
- ❌ 跳过索引直接读取代码文件

## 三、索引生成协议

### 触发条件
用户输入 "Traject 初始化" 时触发。

### 执行步骤

**阶段 1：初始化计划**
1. 创建 `.traject/` 目录和子目录
2. 创建 `exclude.yaml`，包含默认排除列表
3. 扫描根目录，向用户确认排除列表（只询问一级目录）
4. 生成完整的遍历计划，写入 `plan/traversal-state.json`
5. 向用户报告：「已生成索引计划，共需处理 X 个目录。是否开始生成？」

**阶段 2：分步生成**
1. 从 `pending_dirs` 中取出一批目录（建议 5-10 个）
2. 对每个目录：
   a. 扫描其直接子目录和文件
   b. 生成对应的 `.traject.md` 文件
   c. 记录生成状态
3. 每完成一批，更新 `traversal-state.json`
4. 询问用户：「已完成 X/Y 个目录，是否继续？」
5. 重复直到全部完成

**阶段 3：完成收尾**
1. 生成根目录 `.traject/manifest.md`
2. 报告：「✅ Traject 索引全部生成完成！共 X 个目录，Y 个文件已索引。」

### 恢复机制
当用户输入 "Traject 继续" 时：
1. 读取 `plan/traversal-state.json`
2. 向用户报告当前进度
3. 从断点继续生成

## 四、索引更新协议

### 触发条件
- 用户说 "Traject 更新"
- 用户要求提交代码时（自动触发）

### 执行步骤
1. 检测变更文件（通过 `git status` 或用户描述）
2. 定位变更文件所在目录
3. 仅更新这些目录及其父级目录的 `.traject.md`
4. 若新增/删除文件：更新文件列表
5. 若新增/删除子目录：更新子目录列表 + 父级索引
6. 报告更新摘要

## 五、.traject.md 文件规范

每个目录下的 `.traject.md` 必须遵循以下格式：

- 标题：`# 目录索引: [从根目录开始的相对路径]`
- 职责：`## 职责` + 一句话描述
- 子目录：`## 子目录` + 表格（目录名、职责）
- 文件：`## 文件` + 表格（文件名、职责、关键符号）
- 依赖：`## 依赖`（可选）

**强制约束**：
- 每个 `.traject.md` 不超过 30 行
- 每个文件描述不超过 10 个字
- 每个文件最多列出 3 个关键符号
- 只描述**直接子级**，禁止跨级

## 六、命令速查

| 用户指令 | 执行动作 |
|---------|---------|
| `Traject 初始化` | 首次生成全部索引（分步执行） |
| `Traject 加载` | 新会话加载项目概览 |
| `Traject 更新` | 增量更新索引 |
| `Traject 继续` | 恢复中断的生成任务 |
```