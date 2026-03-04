# Skill Hub

个人技能集合，沉淀工作中积累的技术方案、工具技巧和方法论。

## 目录结构

```
SkillHub/
├── skills/              # 所有 Skill 文件（按目录区分）
│   └── {skill-name}/    # 每个 Skill 一个独立目录
├── skill-manager        # Skill 安装工具
├── skill-sync           # 一键同步脚本
├── README.md            # 项目说明
├── SKILLS.md            # Skill 索引
└── .gitignore
```

## 快速开始

### 1. 安装 skill-manager

#### 方式一：通过提示词安装（推荐，无需克隆项目）

将以下提示词复制给 AI 助手（如 Cursor、Codex 等），即可自动完成安装：

```
请通过 https://github.com/PipeDog/PipeDog-SkillHub/blob/main/README.md 中所提供的方案在全局安装 skill-manager，安装成功后验证 skill-manager 的功能是否可以正常使用，并告知用户安装结果
```

> **安装路径说明**：通过提示词安装时，skill-manager 将默认 clone 并安装到用户目录下的 `.skill-manager` 目录（隐藏目录，以 `.` 为前缀），即 `~/.skill-manager`。

#### 方式二：手动安装（需先克隆项目）

```bash
# 1. 克隆项目
git clone https://github.com/PipeDog/PipeDog-SkillHub.git
cd PipeDog-SkillHub

# 2. 将项目添加到 PATH（将 /path/to 替换为你的实际路径）
echo 'export PATH="/path/to/PipeDog-SkillHub:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

> 例如：`${HOME}/Desktop/PipeDog-SkillHub` 或绝对路径 `/Users/leiliang/Desktop/PipeDog-SkillHub`

### 2. 使用 skill-manager 安装 Skills

```bash
# 列出所有 Skills
skill-manager list

# 搜索 Skill
skill-manager search flutter

# 安装 Skill 到项目
skill-manager install flutter-architecture

# 安装到指定目录
skill-manager install flutter-architecture docs/

# 全局安装
skill-manager install-global flutter-architecture
```

### 3. 添加新 Skill

在 `skills/` 目录下创建新的文件夹：

```
skills/
└── your-skill-name/
    ├── SKILL.md         # Skill 主文件（必需）
    └── references/      # 参考资料（可选）
        └── *.md
```

### 4. 同步到 GitHub

```bash
./skill-sync "feat: 添加 XXX Skill"
```

## Skill 目录规范

每个 Skill 建议包含：

```
skill-name/
├── SKILL.md         # 必需：Skill 核心内容
└── references/      # 可选：参考资料
    └── *.md
```

## 示例 Skills

- [flutter-architecture](./skills/flutter-architecture/) - Flutter 四层组件化 + MVVM 项目架构规范
- [skill-manager](./skills/skill-manager/) - Skill 安装工具

## 在其他项目中使用 Skill

### 方式一：使用 skill-manager（推荐）

```bash
# 安装到当前项目
skill-manager install flutter-architecture

# 安装到指定目录
skill-manager install flutter-architecture docs/
```

### 方式二：直接复制

```bash
cp -r PipeDog-SkillHub/skills/flutter-architecture/ ./your-project/docs/
```

### 方式三：Git Submodule

```bash
git submodule add https://github.com/PipeDog/PipeDog-SkillHub.git third_party/SkillHub
```

## 搜索 Skill

```bash
# 使用 skill-manager
skill-manager search <关键词>

# 使用 grep
grep -r "<关键词>" skills/
```

## 提交规范

| 类型    | 说明                 |
| ------- | -------------------- |
| `feat:` | 新增 Skill           |
| `fix:`  | 修复/更新 Skill 内容 |
| `docs:` | 文档更新             |

## License

MIT
