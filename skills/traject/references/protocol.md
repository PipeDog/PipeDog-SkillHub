# Traject 命令协议

本文档定义 4 个命令的完整执行步骤、完成标准、校验逻辑和边界情况处理。

---

## 一、命令：Traject 初始化

### 前置条件

- 当前目录为项目根目录
- 项目根目录可写
- 排除 Profile 位于 `~/.claude/skills/traject/references/excludes/`
- AI 启发式判断规则位于 `~/.claude/skills/traject/references/heuristics.md`

### 阶段 1：规划（Plan）

#### 步骤 1.1 — 创建目录结构

1. 创建 `.traject/` 目录
2. 创建 `.traject/project/` 子目录
3. 创建 `.traject/plan/` 子目录
4. **检测项目类型并合并 Profile**：
   a. 扫描项目根目录的标志文件，确定项目类型：
      | 标志文件 | 项目类型 | 加载 Profile |
      |---------|---------|-------------|
      | `pubspec.yaml` | Flutter / Dart | `common.yaml` + `flutter.yaml` |
      | `package.json` | Node.js / 前端 | `common.yaml` + `node.yaml`（未来） |
      | `go.mod` | Go | `common.yaml` |
      | `Cargo.toml` | Rust | `common.yaml` |
      | `requirements.txt` / `pyproject.toml` | Python | `common.yaml` |
      | 无匹配 | 通用项目 | `common.yaml` |
   b. 多个标志文件同时存在时，合并所有匹配的 Profile（去重）
   c. 读取 `~/.claude/skills/traject/references/heuristics.md` 加载 AI 判断规则
5. 将合并后的排除配置写入 `.traject/exclude.yaml`（包含 `activated_profiles` 记录）

**完成标准**：`.traject/`、`.traject/project/`、`.traject/plan/` 目录存在，`.traject/exclude.yaml` 已生成并包含激活的 profile 列表

#### 步骤 1.2 — 扫描一级目录并确认排除

1. 列出项目根目录下所有一级目录（排除 `.traject/` 自身）
2. 读取 `.traject/exclude.yaml` 中的 `exclude_dirs`、`exclude_paths`、`exclude_files`
3. **执行 AI 启发式判断**（按 `references/heuristics.md` 的规则）：
   a. 对每个非 Profile 排除的一级目录，执行目录级硬规则检查（heuristics.md 第 2 节）
   b. 对未匹配硬规则的目录，执行目录级软规则检查（heuristics.md 第 3 节）
   c. 按置信度分级：高置信度（≥80%）→ 中置信度（50%-79%）→ 低置信度（<50%，保留）
4. 将一级目录分为三组，向用户展示：

```
🔍 检测到项目类型：Flutter + Node.js

🚫 自动排除（Profile 规则 + 高置信度 AI 判断）：
  - .dart_tool/       （Flutter 工具缓存）
  - Pods/             （CocoaPods 依赖）
  - node_modules/     （Node 依赖）
  - .git/             （版本控制）
  - linux/            （Flutter Linux 平台，自动生成）
  - elinux/           （Flutter Embedded Linux 平台）

⚠️ 建议排除（中置信度 AI 判断）：
  - test/             （测试代码，非核心实现）
  - docs/             （文档，非代码实现）
  - scripts/          （工具脚本，非核心代码）

📂 将建立索引的目录：
  - lib/              （Flutter 主代码）
  - assets/           （资源文件）
  - example/          （示例代码，lib/ 部分将索引）

是否确认？你可以：
  - 直接确认（回车）
  - 将目录从 🚫 移到 📂：「索引 test/」
  - 将目录从 📂 移到 🚫：「排除 assets/」
  - 将目录从 ⚠️ 移到 🚫：「排除 scripts/」
```

5. 特别标注：`test/` 和 `docs/` 默认在 Profile 中排除，但会在此处明确提示用户可调整
6. 根据用户反馈更新 `exclude.yaml` 和 `user_confirmed` 标记

**完成标准**：用户已确认排除列表，`exclude.yaml` 中 `user_confirmed: true`

#### 步骤 1.3 — 全量扫描与排序

1. 使用 `find` 命令扫描所有目录，排除 `exclude_dirs` 和 `exclude_paths` 中的目录：

```bash
# 构建排除条件（同时处理目录名匹配和路径匹配）
# 1. 目录名排除：-not -path '*/dirname/*'
# 2. 路径排除：-not -path './specific/path/*'
find . -type d \
  -not -path '*/node_modules/*' \
  -not -path '*/\.git/*' \
  -not -path './example/macos/Runner/*' \
  # ... 更多排除条件
  | sort
```

2. 对每个目录计算 depth（相对于项目根目录的层级数）
3. 按 depth 降序排列（叶子优先），同 depth 按路径字母序
4. 构建 `pending_dirs` 数组：

```json
[
  {"path": "lib/components/buttons/", "depth": 3, "status": "pending"},
  {"path": "lib/components/forms/", "depth": 3, "status": "pending"},
  {"path": "lib/components/", "depth": 2, "status": "pending"},
  {"path": "lib/utils/", "depth": 2, "status": "pending"},
  {"path": "lib/", "depth": 1, "status": "pending"}
]
```

**完成标准**：`pending_dirs` 包含所有非排除目录，按拓扑序排列

#### 步骤 1.4 — 写入遍历计划

1. 生成 `traversal-state.json`，填入所有字段：

```json
{
  "version": "1.0",
  "created_at": "",
  "last_updated": "",
  "activated_profiles": ["common"],
  "total_dirs": 0,
  "processed_dirs": 0,
  "pending_dirs": [],
  "failed_dirs": [],
  "current_batch": 0,
  "batch_size": 7,
  "excluded_dirs": [],
  "ai_excluded": [],
  "ai_suggested": [],
  "status": "not_started",
  "status_note": "等待用户确认排除列表"
}
```

2. 写入 `.traject/plan/traversal-state.json`

**完成标准**：`traversal-state.json` 存在，`status: "plan_ready"`，`total_dirs` > 0

#### 步骤 1.5 — 报告并等待确认

向用户报告：

```
✅ 索引计划已生成

📊 统计：
  - 待索引目录：X 个
  - 排除目录：Y 个
  - 批次大小：7 个/批
  - 预计批次：Z 批

是否开始生成索引？
```

**完成标准**：用户确认开始生成

### 阶段 2：分批生成（Generate）

#### 步骤 2.1 — 取出一批目录

1. 更新 `traversal-state.json`：`status: "generating"`
2. 从 `pending_dirs` 头部取出 `batch_size`（默认 7）个目录
3. `current_batch += 1`

**完成标准**：当前批次目录列表已确定

#### 步骤 2.2 — 处理每个目录

对批次中的每个目录，按 depth 降序处理：

**情况 A：叶子目录**（该目录下无子目录，或所有子目录均被排除）

1. 使用 `ls` 列出目录下的文件
2. 过滤掉 `exclude_files` 中的文件（文件名模式匹配）
3. 过滤掉二进制文件（扩展名匹配，见 heuristics.md 第 4.1 节）
4. 过滤掉自动生成文件（文件名模式匹配，见 heuristics.md 第 4.2 节）
5. 对剩余文件，执行 **文件级 AI 判断**（heuristics.md 第 5 节）：
   - 读取文件头部（前 20 行）
   - 检查是否包含 `@generated`、`auto-generated`、`DO NOT EDIT` 等标记
   - 如命中 → 跳过索引（标注为自动生成）
   - 如未命中 → 正常索引
6. 对需要索引的文件：
   - 读取文件内容（对于大文件只读前 100 行 + 后 50 行）
   - 提取关键符号（函数名、类名、导出）
   - 归纳职责（≤10 字）和摘要（15-25 字）
7. 生成 `.traject.md`，写入 `.traject/project/{当前目录路径}/.traject.md`（如目录不存在则 `mkdir -p` 创建）
8. 标记 `status: "processed"`

**情况 B：非叶子目录**（有子目录需要索引）

1. 列出该目录下的子目录（排除 `exclude_dirs` 和 `exclude_paths`）
2. 对每个子目录，读取其 `.traject/project/{子目录路径}/.traject.md` 中的「职责」行
3. 列出该目录下的直接文件（排除子目录、`exclude_files`、二进制文件、自动生成文件）
4. 对直接文件，执行文件级 AI 判断（同情况 A 步骤 5），然后读取并提取关键符号
5. 汇总子目录职责 + 直接文件信息，生成 `.traject/project/{当前目录路径}/.traject.md`
6. 标记 `status: "processed"`

**关键约束**：非叶子目录的「职责」必须从子目录的 `.traject/project/{子目录路径}/.traject.md` 汇总而来，不得直接读取孙子目录的源码。

**完成标准**：每个目录的 `.traject.md` 已生成，内容符合格式规范

#### 步骤 2.3 — 批次校验

每批处理完成后，对生成的 `.traject.md` 进行校验：

1. 行数检查：`wc -l` 每文件 ≤ 35 行
2. 格式检查：包含必需的 `# 目录索引`、`## 职责` 标题
3. 描述长度检查：职责 ≤ 10 字，摘要软约束 15-25 字
4. 符号数检查：每个文件关键符号 ≤ 3 个
5. 文件级 AI 判断复核：检查是否有遗漏的自动生成文件（抽样检查 10% 的已索引文件头部）

如发现校验失败：
- 标记该目录为 `failed`
- 加入 `failed_dirs` 列表
- 下一批重试

**完成标准**：本批次所有文件校验通过，或失败目录已记录

#### 步骤 2.4 — 更新状态

1. 更新 `traversal-state.json`：
   - 将已处理目录从 `pending_dirs` 移除
   - `processed_dirs += 本批处理数`
   - `last_updated = 当前时间`
   - 如有失败，加入 `failed_dirs`

**完成标准**：`traversal-state.json` 已更新

#### 步骤 2.5 — 报告并询问

```
📊 进度：X/Y 个目录已完成（Z%）

✅ 本批完成：
  - src/components/buttons/  （3 个文件）
  - src/components/forms/    （5 个文件）
  - src/components/          （2 个子目录，1 个文件）

❌ 失败（下批重试）：
  - （无）

还剩 Z 个目录，是否继续？
```

**完成标准**：用户确认继续或暂停

#### 步骤 2.6 — 循环

重复步骤 2.1-2.5，直到 `pending_dirs` 为空。

如果 `failed_dirs` 非空（重试后仍失败），向用户报告失败列表并询问处理方式。

**完成标准**：`pending_dirs` 为空，`status: "generation_complete"`

### 阶段 3：收尾（Finalize）

#### 步骤 3.1 — 生成 manifest.md

1. 列出项目根目录下的一级目录（排除 `exclude_dirs`）
2. 对每个一级目录，读取其 `.traject/project/{一级目录路径}/.traject.md` 中的「职责」行
3. 汇总统计信息（总目录数、总文件数）
4. 按模板生成 `manifest.md`，写入 `.traject/manifest.md`

**完成标准**：`manifest.md` 存在，包含所有一级目录的索引

#### 步骤 3.2 — 生成 AGENTS.md

1. 读取 `~/.claude/skills/traject/references/templates.md` 中的 AGENTS.md 模板
2. 直接写入项目根目录 `AGENTS.md`

**完成标准**：`AGENTS.md` 存在

#### 步骤 3.3 — 生成/更新 README.md

1. 检查 `README.md` 是否存在
2. 若不存在，创建 README.md，包含 Traject 使用说明
3. 若存在，在末尾追加 Traject 使用说明（如尚未包含）

**完成标准**：README.md 包含 Traject 使用说明

#### 步骤 3.4 — 最终报告

```
✅ Traject 索引全部生成完成！

📊 最终统计：
  - 索引目录：X 个
  - 索引文件：Y 个
  - 生成 .traject.md：X 个
  - 排除目录：Z 个

📌 已生成文件：
  - AGENTS.md（AI 自动加载）
  - .traject/manifest.md（根索引）
  - .traject/exclude.yaml（排除配置）
  - .traject/plan/traversal-state.json（状态记录）
  - X 个 .traject.md（存储于 .traject/project/ 目录下）

💡 使用提示：
  - 新会话：输入 "Traject 加载" 快速了解项目
  - 代码变更：输入 "Traject 更新" 更新索引
  - 日常提问：直接提问，AI 按索引自动定位
```

**完成标准**：`traversal-state.json` 中 `status: "completed"`

---

## 二、命令：Traject 加载

### 步骤 2.1 — 检测索引

1. 检查 `.traject/manifest.md` 是否存在

**完成标准**：已确认 manifest.md 存在与否

### 步骤 2.2 — 加载索引（若存在）

1. 读取 `.traject/manifest.md`
2. 读取 `.traject/exclude.yaml`
3. 提取关键信息：
   - 项目名称/概述
   - 一级目录列表及职责
   - 索引统计（目录数、文件数、生成时间）
   - 排除的目录
4. 向用户报告：

```
📋 项目概览（Traject 索引）

项目：[项目名称]
一级目录：
  - src/          — 源码目录
  - config/       — 配置目录
  - docs/         — 文档目录

📊 索引统计：X 个目录，Y 个文件 | 生成于 [时间]

请提出具体问题，我会按索引定位到相应代码。
```

**完成标准**：已报告项目概览，等待用户提问

### 步骤 2.3 — 提示缺失（若不存在）

向用户报告：

```
⚠️ 未检测到 Traject 索引。

输入 "Traject 初始化" 开始建立项目索引。

建立索引后：
- 新会话可快速加载项目概览
- AI 按索引定位代码，无需通读项目
- 大幅节省上下文窗口
```

---

## 三、命令：Traject 更新

### 步骤 3.1 — 检测变更

1. 运行 `git status --porcelain` 获取变更文件列表
2. 如果项目无 git 仓库，退化为全量扫描（告知用户）

**完成标准**：已获取变更文件列表

### 步骤 3.2 — 构建受影响目录集合

1. 从变更文件列表提取文件路径
2. 对每个变更文件，取其所在目录路径
3. 去重，得到「直接受影响目录」集合
4. 扩展：对每个直接受影响目录，沿路径向上，将其所有祖先目录加入集合

```
示例：
变更文件：src/components/buttons/PrimaryButton.tsx

直接受影响：src/components/buttons/
向上冒泡：  src/components/ → src/
```

5. 按 depth 降序排列（叶子优先）

**完成标准**：受影响目录集合已构建，按拓扑序排列

### 步骤 3.3 — 重新生成索引

对受影响目录集合中的每个目录，按 depth 降序处理：

1. **叶子目录**：重新读取源码文件 → 重新生成 `.traject/project/{路径}/.traject.md`
2. **非叶子目录**：重新读取子级 `.traject/project/{子目录路径}/.traject.md` → 重新生成 `.traject/project/{路径}/.traject.md`
3. 比较新旧 `.traject/project/{路径}/.traject.md` 内容：
   - 内容相同 → 停止向上冒泡（该路径的父级无需更新）
   - 内容不同 → 继续向上冒泡

**完成标准**：所有受影响目录的 `.traject/project/{路径}/.traject.md` 已更新

### 步骤 3.4 — 处理新增/删除

1. 检测是否有新增目录（不在索引中，但不在排除列表）
2. 检测是否有删除目录（在索引中，但目录已不存在）
3. 新增目录：按叶子/非叶子逻辑生成 `.traject/project/{路径}/.traject.md`
4. 删除目录：删除 `.traject/project/{路径}/.traject.md`，更新父级索引

**完成标准**：新增/删除目录已处理

### 步骤 3.5 — 更新 manifest.md

1. 如果一级目录的 `.traject/project/{一级目录路径}/.traject.md` 有变化，更新 `manifest.md`
2. 更新 `generated_at` 时间戳

**完成标准**：manifest.md 已同步

### 步骤 3.6 — 报告更新摘要

```
✅ Traject 索引已更新

📝 变更文件：N 个
📂 更新目录：M 个
  - src/components/buttons/  （文件变更）
  - src/components/          （子级索引变更）
  - src/                     （子级索引变更）

⏱️ 更新完成于 [时间]
```

---

## 四、命令：Traject 继续

### 步骤 4.1 — 读取状态

1. 读取 `.traject/plan/traversal-state.json`
2. 检查 `status` 字段

**完成标准**：已读取状态文件

### 步骤 4.2 — 判断可恢复性

| status | 可恢复？ | 处理 |
|--------|---------|------|
| `plan_ready` | ✅ | 从阶段 2 开始（进入分批生成） |
| `generating` | ✅ | 从 `pending_dirs` 继续生成 |
| `generation_complete` | ⚠️ | 提示已完成生成，询问是否重新生成 |
| `completed` | ❌ | 提示已完成，如需重新生成请用「Traject 初始化」 |
| `not_started` | ❌ | 提示尚未开始，请用「Traject 初始化」 |

### 步骤 4.3 — 报告进度

```
📊 恢复进度

已完成：X/Y 个目录（Z%）
待处理：W 个目录
失败重试：V 个目录
当前批次：第 N 批

从断点继续？(Y/n)
```

### 步骤 4.4 — 继续生成

用户确认后，从 `pending_dirs` 的当前位置继续执行「阶段 2：分批生成」的所有步骤。

---

## 五、校验器逻辑

### 格式校验

对每个 `.traject.md` 执行以下检查：

```bash
# 1. 行数检查
lines=$(wc -l < "$file")
if [ "$lines" -gt 35 ]; then
  echo "FAIL: $file 有 $lines 行，超过 35 行限制"
fi

# 2. 必需标题检查
grep -q "^# 目录索引:" "$file" || echo "FAIL: $file 缺少标题"
grep -q "^## 职责" "$file" || echo "FAIL: $file 缺少职责节"

# 3. 描述长度检查（提取「文件」表格中的职责和摘要列）
# 提取 | filename | desc | summary | symbols | 中的 desc 和 summary 列，检查长度
```

### 完整性校验

```bash
# 检查所有非排除目录是否都有对应的 .traject.md
find . -type d -not -path '*/\.*' -not -path '*/node_modules/*' | while read dir; do
  if [ ! -f ".traject/project/$dir/.traject.md" ]; then
    echo "MISSING: $dir"
  fi
done
```

### 一致性校验

```bash
# 检查父目录的「子目录」列表是否与实际子目录一致
# 父目录 .traject/project/{路径}/.traject.md 中列出的子目录应等于实际存在的子目录
```

---

## 六、边界情况处理

### 6.1 空目录

目录下无文件且无子目录：

```markdown
# 目录索引: path/to/empty/

## 职责
空目录（预留）

## 文件
（无）
```

### 6.2 文件过多（超过 35 行容量）

当目录下文件数超过 35 行容量时：

1. 优先列出「核心文件」（入口文件、命名的导出文件、配置文件）
2. 其余文件归入表格末尾：

```markdown
| ... | （其余 N 个文件） | - | - |
```

3. 在职责描述中注明「包含 N 个文件」

### 6.3 无 git 仓库

`Traject 更新` 命令退化为：

1. 告知用户：「当前项目无 git 仓库，将执行全量扫描」
2. 执行全量扫描（类似初始化，但保留已有索引）
3. 比较新旧索引，报告差异

### 6.4 二进制文件

检测方式：
- 扩展名匹配：见 `references/heuristics.md` 第 4.1 节完整列表
- `file` 命令检测 MIME 类型兜底

处理：跳过，不列入索引

### 6.5 自动生成文件

检测方式（按优先级）：
1. 文件名模式匹配：见 `references/heuristics.md` 第 4.2 节
2. 文件头部标记检测：读取前 20 行，检查 `@generated`、`auto-generated`、`DO NOT EDIT` 等标记
3. 代码生成器特征检测：文件头部包含生成器名称（如 `generated by json_serializable`）

处理：跳过，不列入索引。在 `traversal-state.json` 的 `ai_excluded` 字段中记录

### 6.6 AI 判断与 Profile 规则冲突

当 AI 判断认为 Profile 规则可能有问题时：
- 不自动覆盖 Profile 规则
- 在确认环节提示用户：「⚠️ `build/` 目录被 Profile 排除，但检测到其中包含手写脚本，建议从排除列表中移除」
- 用户做最终决定

### 6.7 符号链接

- 跟随符号链接读取目标
- 在路径中标注 `→ [target]`
- 不重复索引（同一个真实路径只索引一次）

### 6.8 排除后无剩余目录

```
⚠️ 排除列表覆盖了所有目录，没有可索引的目录。

请调整排除列表，至少保留一个源码目录。
```

### 6.9 大文件（>1000 行）

- 只读取前 100 行 + 后 50 行提取关键符号
- 不读取完整文件内容（节省上下文）

### 6.10 会话中断恢复

- 每次处理完一批后立即更新 `traversal-state.json`
- 每批处理前检查 `traversal-state.json` 是否存在
- 如果存在且 `status: "generating"`，自动进入恢复模式

### 6.11 项目类型检测失败

当项目根目录无可识别的标志文件时：
- 仅加载 `common.yaml` Profile
- 向用户提示：「未检测到已知项目类型，仅加载通用排除规则。如需添加平台特定排除，请手动编辑 `.traject/exclude.yaml`」
- AI 判断仍正常执行（补充 Profile 覆盖不到的目录）