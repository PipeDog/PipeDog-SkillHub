# Skill Hub

个人技能集合，沉淀工作中积累的技术方案、工具技巧和方法论。

## 目录结构

```
SkillHub/
├── skills/              # 所有 Skill 文件（扁平化存储，按目录区分）
│   └── {skill-name}/    # 每个 Skill 一个独立目录
├── skill-sync           # 一键同步脚本
├── README.md            # 项目说明
├── SKILLS.md            # Skill 索引
└── .gitignore
```

## 快速开始

### 1. 克隆项目

```bash
git clone git@github.com:PipeDog/PipeDog-SkillHub.git
cd PipeDog-SkillHub
```

### 2. 添加新 Skill

在 `skills/` 目录下创建新的文件夹，每个 Skill 独立存放：

```
skills/
└── your-skill-name/
    ├── SKILL.md         # Skill 主文件（必需）
└── references/      # 参考资料（可选）
    └── *.md
```

### 3. 同步到 GitHub

添加或更新 Skill 后，一键推送：

```bash
# 默认提交信息
./skill-sync

# 自定义提交信息
./skill-sync "feat: 添加 XXX Skill"
```

## Skill 目录规范

每个 Skill 建议包含：

```
skill-name/
├── SKILL.md         # 必需：Skill 核心内容
└── references/      # 可选：参考资料、文档链接
    └── *.md
```

## 示例 Skill

- [flutter-architecture](./skills/flutter-architecture/) - Flutter 四层组件化 + MVVM 项目架构规范

## 在项目中使用 Skill

### 方式一：直接复制（推荐）

将需要的 Skill 文件复制到你的项目中：

```bash
# 复制整个 Skill 目录
cp -r PipeDog-SkillHub/skills/flutter-architecture/ ./your-project/docs/
```

### 方式二：Git Submodule

将 SkillHub 作为子模块引入项目：

```bash
# 添加为子模块
git submodule add git@github.com:PipeDog/PipeDog-SkillHub.git third_party/SkillHub

# 更新子模块
git submodule update --init --recursive

# 之后更新
cd third_party/SkillHub && git pull && cd -
```

### 方式三：Git Subtree

```bash
# 共享 Skill 到项目
git subtree add --prefix=docs/skills/PipeDog-SkillHub git@github.com:PipeDog/PipeDog-SkillHub main --squash

# 后续更新
git subtree pull --prefix=docs/skills/PipeDog-SkillHub git@github.com:PipeDog/PipeDog-SkillHub main --squash
```

## 搜索 Skill

- [flutter-architecture](./skills/flutter-architecture/) - Flutter 四层组件化 + MVVM 项目架构规范

## 搜索 Skill

```bash
# 按文件名搜索
ls skills/ | grep <关键词>

# 按内容搜索
grep -r "<关键词>" skills/

# 搜索特定 Skill
ls skills/*/
```

## 提交规范

使用 `skill-sync` 脚本时，遵循以下提交类型：

| 类型 | 说明 |
|------|------|
| `feat:` | 新增 Skill |
| `fix:` | 修复/更新 Skill 内容 |
| `docs:` | 文档更新 |

## License

MIT
