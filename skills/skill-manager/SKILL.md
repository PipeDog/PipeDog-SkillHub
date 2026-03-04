# Skill Manager - Skill 安装工具

> Skill 管理器，用于安装 SkillHub 中的 Skills 到项目或全局

## 简介

`skill-manager` 是一个命令行工具，用于从 SkillHub 仓库安装 Skills 到你的项目或全局环境。

## 功能

- 列出所有可用的 Skills
- 搜索 Skills
- 安装 Skill 到当前项目
- 安装 Skill 到全局

## 安装

### 方式一：克隆项目

```bash
git clone git@github.com:PipeDog/PipeDog-SkillHub.git
cd PipeDog-SkillHub
```

### 方式二：添加到 PATH（推荐）

将项目克隆到常用目录，并添加到 PATH：

```bash
# 在 ~/.bashrc 或 ~/.zshrc 中添加
export PATH="/path/to/PipeDog-SkillHub:$PATH"
```

然后重启终端或运行 `source ~/.zshrc`

## 使用方式

### 列出所有 Skills

```bash
skill-manager list
```

### 搜索 Skill

```bash
skill-manager search <关键词>
# 示例
skill-manager search flutter
```

### 安装 Skill 到项目

```bash
skill-manager install <skill-name>
# 示例
skill-manager install flutter-architecture
```

指定目标目录（可选）：

```bash
skill-manager install flutter-architecture docs/
```

### 安装 Skill 到全局

```bash
skill-manager install-global <skill-name>
# 示例
skill-manager install-global flutter-architecture
```

全局安装位置：`~/.skill-manager/skills/`

## 可用 Skills

| Skill | 描述 |
|-------|------|
| flutter-architecture | Flutter 四层组件化 + MVVM 项目架构规范 |
| skill-manager | Skill 安装工具（当前） |

## 常用场景

### 在新项目中快速添加架构规范

```bash
# 克隆后进入项目目录
cd my-new-flutter-project

# 安装 Flutter 架构规范
skill-manager install flutter-architecture docs/
```

### 全局保留常用技能

```bash
# 全局安装，随时使用
skill-manager install-global flutter-architecture

# 在任意项目中使用
cd any-project
ls ~/.skill-manager/skills/
```

## 更新 SkillManager

```bash
cd PipeDog-SkillHub
git pull origin main
```

## 技术实现

- 纯 Bash 脚本，无外部依赖
- 使用 `cp -r` 复制文件
- 全局安装使用 `~/.skill-manager/` 目录

## 更新日志

- `2026-03-04` - 初始版本
