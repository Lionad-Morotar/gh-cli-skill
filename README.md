# gh-cli-skill

GitHub CLI (gh) comprehensive reference skill for Claude.

## 安装

```bash
npx skills add https://github.com/Lionad-Morotar/gh-cli-skill
```

## 使用

```sh
/gh-cli {你的要求，如"创建一个新的 GitHub 仓库"}
```

如果你的 IDE 不支持 SlashCommand，那么为了获得最可靠的结果，需要提示词前加上前缀，比如：

```plaintext
使用 gh {你的要求，如"创建一个新的 GitHub 仓库"}
```

这会明确触发技能并确保 AI 遵循文档化的模式。如果不加前缀，技能触发可能不一致，具体取决于你的提示词与技能描述关键词的匹配程度。
