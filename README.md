# claude-skills

[Claude Code](https://claude.com/claude-code) skill 收藏（@bighb）。

## Skills

| Skill | 用途 | 触发 |
|---|---|---|
| [imagine-router](./imagine-router) | 配图调度员——把"配图 / 出图 / 画个图"等模糊需求路由到正确的 baoyu-* skill | 用户说"配图" / "画个图" / "出张封面" 等，或显式 `/imagine-router` |

## 安装单个 skill

把整个 skill 目录拷到 `~/.claude/skills/` 即可，Claude Code 下次启动时会自动发现。

**macOS / Linux**

```bash
git clone https://github.com/bighb/claude-skills.git
cp -r claude-skills/imagine-router ~/.claude/skills/
```

**Windows PowerShell**

```powershell
git clone https://github.com/bighb/claude-skills.git
Copy-Item -Recurse claude-skills\imagine-router $env:USERPROFILE\.claude\skills\
```

或只下载某一个 skill 的 SKILL.md（适合无 git 环境）：

```bash
mkdir -p ~/.claude/skills/imagine-router
curl -fsSL https://raw.githubusercontent.com/bighb/claude-skills/main/imagine-router/SKILL.md \
  -o ~/.claude/skills/imagine-router/SKILL.md
```

## imagine-router 的额外依赖

这个 skill 自己只是调度员，真正出图靠下游的 [baoyu-skills](https://github.com/JimLiu/baoyu-skills) 套装。完整链路需要：

1. **安装 baoyu-skills**（提供 7 个出图 skill）

   ```bash
   npx --yes skills add jimliu/baoyu-skills -g -a claude-code \
     --skill baoyu-imagine --skill baoyu-article-illustrator --skill baoyu-cover-image \
     --skill baoyu-xhs-images --skill baoyu-comic --skill baoyu-diagram --skill baoyu-infographic \
     --copy -y
   ```

2. **配置图像 provider**：在 `~/.baoyu-skills/baoyu-imagine/EXTEND.md` 写入 `default_provider` 等偏好（schema 见 [baoyu-skills 仓库](https://github.com/JimLiu/baoyu-skills)）。支持 Google Gemini、OpenAI、Azure、OpenRouter、DashScope（阿里通义万象）、Z.AI、MiniMax、Replicate、Volcengine Seedream（豆包）等 provider。

3. **设环境变量**：根据 provider 设对应 API key，例如：
   - `ARK_API_KEY` for 火山引擎 Seedream
   - `GOOGLE_API_KEY` for Gemini
   - `OPENAI_API_KEY` for GPT Image
   - 等等

## 关于 Claude Code skill 机制

每个 skill 由一个 `SKILL.md` 文件定义——YAML frontmatter（`name`、`description`）描述触发条件，正文是给 Claude 的执行指令。Claude 在每次响应前根据 description 判断是否激活该 skill。

详见 [Claude Code 官方文档](https://docs.claude.com/claude-code)。

## License

MIT
