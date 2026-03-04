# Skill Hub

个人技能集合，沉淀工作中积累的技术方案、工具技巧和方法论。

## 目录结构

```
SkillHub/
├── skills/              # 所有 Skill 文件（扁平化存储）
│   └── *.md
├── templates/           # Skill 模板
│   └── skill-template.md
├── README.md            # 项目说明
├── SKILLS.md            # Skill 索引（自动生成）
└── .gitignore
```

## 如何添加新 Skill

1. 复制 `templates/skill-template.md` 到 `skills/` 目录
2. 重命名为描述性的文件名（如 `git-undo-commit.md`）
3. 填写模板内容，确保包含：
   - 清晰的标题
   - 准确的标签（用于搜索）
   - 分类标签：`开发` / `工具` / `方法论` / `其他`
   - 核心内容和代码示例
4. 更新 `SKILLS.md` 添加索引

## 标签规范

常用标签参考：

| 标签 | 含义 |
|------|------|
| `git` | Git 相关 |
| `shell` | 命令行/Shell |
| `mac` | macOS 相关 |
| `ide` | 开发工具/IDE |
| `debug` | 调试技巧 |
| `performance` | 性能优化 |
| `config` | 配置相关 |
| `workflow` | 工作流 |
| `template` | 模板/脚手架 |

## 搜索 Skill

在终端中快速搜索：

```bash
# 按文件名搜索
ls skills/ | grep <关键词>

# 按内容搜索
grep -r "<关键词>" skills/

# 按标签筛选
grep -l "标签" skills/*.md
```

## License

MIT
