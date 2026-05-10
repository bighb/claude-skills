---
name: imagine-router
description: Image generation request router. Use this whenever the user asks to generate, create, draw, illustrate, or make any kind of image — including covers, hero images, article illustrations, social-media cards (小红书/微信), diagrams (flowchart/architecture/sequence/mind map), infographics, comics, picture books, posters, or free-form prompts. Triggers on Chinese phrases "配图" "出图" "生图" "做图" "画图" "画一张" "画个图" "封面" "题图" "小红书图" "图卡" "流程图" "架构图" "时序图" "思维导图" "信息图" "可视化" "漫画" "绘本" "插画" "故事图" and English "illustrate" "draw" "generate image" "make a cover" "design a poster". The user explicitly built this router to avoid memorizing the seven baoyu-* skill names — so do NOT guess which baoyu-* skill to invoke based on keywords. Always run this router first to interview the user, confirm intent, then dispatch. Skip this router only when the user has named a specific baoyu-* skill themselves (e.g., they typed "/baoyu-cover-image" or "用 baoyu-diagram 画").
---

# imagine-router — 配图调度员

用户搭建这个 skill 的动机：**记不住 7 个 baoyu-* skill 名字**。我（执行这个 skill 的 Claude）的工作是把模糊的"配图"需求翻译成具体的下游 skill 调用——通过**一次选项题 + 一次复述确认**完成。

## 触发判断

启动本流程，当：
- 用户说"配图/出图/生图/做图/画图/画一张/画个图/封面/题图/小红书图/图卡/流程图/架构图/时序图/思维导图/信息图/可视化/漫画/绘本/插画/故事图/illustrate/draw/generate image"等含义为"生成图片"的表达
- **且** 用户没有明确点名某个具体 baoyu-* skill

跳过本流程，直接走对应 skill：
- 用户输入了 `/baoyu-cover-image`、`/baoyu-diagram` 等斜杠命令
- 用户原话是"用 baoyu-comic 给我..."、"调 baoyu-imagine 出..."

## 流程（严格按顺序）

### 第 1 步：问意图（一问到底，7 选 1）

用 `AskUserQuestion` 工具，single-select，**只问一个问题、给 7 个场景选项**。不要拆成多轮、不要先问大类再问小类——用户特意要求"一问到底"。

```yaml
question: "想出哪种图？"
header: "类型"
multiSelect: false
options:
  - label: "文章封面 / Hero 图"
    description: "1 张大图，社媒/博客头图。可选 2.35:1、16:9、1:1。下游：baoyu-cover-image"
  - label: "文章正文配图（多张穿插）"
    description: "分析文章结构、自动定位插图位、多张风格统一。下游：baoyu-article-illustrator"
  - label: "小红书图卡 / 微信贴图"
    description: "3:4 竖版多图（1-10 张），社媒种草样式。下游：baoyu-xhs-images"
  - label: "流程图 / 架构图（SVG，零成本）"
    description: "流程、架构、时序、ER、思维导图、时间线。出 SVG，不调图像 API。下游：baoyu-diagram"
  - label: "信息图 / 可视化海报"
    description: "高密度信息大图，21 布局 × 22 风格。下游：baoyu-infographic"
  - label: "知识漫画 / 故事绘本（多页角色一致）"
    description: "教育漫画、传记漫画、绘本，自动建 character sheet 保证多页一致。下游：baoyu-comic"
  - label: "自由出图（直接给 prompt）"
    description: "不走任何模板，原生 prompt 直接出图。下游：baoyu-imagine"
```

### 第 2 步：补一个最小上下文问题（如果还需要）

根据第 1 步选什么，决定补问什么。**只问真正缺的**——如果用户原始消息里已经给了文章路径/主题/prompt，跳过此步。

| 第 1 步选了 | 第 2 步问 |
|---|---|
| cover-image / article-illustrator / xhs-images / infographic / comic | "内容来自哪里？"（A. 文件路径 B. 直接粘贴 C. 现场说主题）|
| diagram | "画什么？一句话描述"（自由文本，不用选项）|
| imagine（自由出图） | "prompt 是什么？"（自由文本）|

如果用户给了文件路径，先 Read 一下确认能读到。读不到就回到这一步重问。

### 第 3 步：复述、等用户点头

用 `AskUserQuestion`，2 选 1（"确认 / 改一下"），把意图复述清楚：

```
理解的需求：
- 类型：[第 1 步选中的类型，比如"文章封面"]
- 内容：[简短概括，不超过 30 字]
- 下游 skill：baoyu-[X]

确认后我就调下游 skill 帮你出。继续吗？
```

```yaml
question: "[上面那段复述]\n\n确认吗？"
header: "确认"
multiSelect: false
options:
  - label: "确认，调下游 skill"
    description: "立即触发 baoyu-[X]，由它接管后续提问（比例、风格、调色板等）"
  - label: "改一下"
    description: "回到第 1 步重选 / 重新描述需求"
```

如果用户选"改一下"，回到第 1 步重新走。

### 第 4 步：调下游 skill

用户点头后，用 `Skill` 工具触发对应 baoyu-* skill。`args` 字段里传一段结构化上下文，让下游 skill 不用从零开始问：

```
内容来源：[文章路径 或 主题描述]
关键约束：[如果用户提了，比如 比例=16:9 / 字数限制 / 风格关键词 / 张数]
其他备注：[用户原话里值得保留的细节，比如目标读者、用途]
```

下游 skill 会接管，按它自己的流程问详细问题（比例、风格、调色板、quality 等）。**本 router 的工作到此结束**——不要在 router 里追加问题、不要试图覆盖下游 skill 的提问流程。

## 路由表（权威）

| 用户场景 | 下游 skill | 备注 |
|---|---|---|
| 文章封面 / Hero / 题图 | `baoyu-cover-image` | 5 维度（type/palette/rendering/text/mood）|
| 文章正文配图（多张） | `baoyu-article-illustrator` | Type × Style × Palette 三维一致性 |
| 小红书图卡 / 微信贴图 / 小绿书 | `baoyu-xhs-images` | 12 风格 × 8 布局 × 3 调色板 |
| 流程图 / 架构图 / 时序图 / 思维导图 / ER 图 / 时间线 | `baoyu-diagram` | **SVG 输出、零 API 成本** |
| 信息图 / 可视化海报 / 高密度信息大图 | `baoyu-infographic` | 21 layout × 22 style |
| 知识漫画 / 教育漫画 / 绘本 / 教程漫画 | `baoyu-comic` | 自动 character sheet |
| 自由出图（任意 prompt，无模板） | `baoyu-imagine` | 底层引擎 |

## 模糊地带的判断规则

下面这些场景容易选错，按规则路由（如果用户已经在第 1 步选了，**用户的选择优先**，下面只在用户犹豫或表达模糊时用）：

- **"给文章配封面 + 多张正文图"** → 当作两个任务串行：先 cover-image 出封面，再 article-illustrator 出正文
- **"故事 + 几张配图"**：
  - 多页、要角色一致 → comic
  - 只是文章里穿插几张 → article-illustrator
- **"做个海报介绍 X"**：
  - 文字密度高、有数据/列表 → infographic
  - 主要靠视觉冲击、文字少 → cover-image
- **"给小红书写攻略"**：
  - 多张卡片串成系列 → xhs-images
  - 只要一张总结图 → infographic
- **"画个图说明 X 怎么工作"**：
  - 有节点/箭头/流程结构 → **diagram 优先**（零成本）
  - 是抽象概念/隐喻/无结构 → imagine 或 infographic

## 不要做的事

- ❌ 不要直接调下游 skill 而跳过第 1 步——本 router 存在的全部意义就是**避免用户被迫记 skill 名**
- ❌ 不要在第 1 步问超过 1 个问题——一问到底 7 选 1 是 router 选定的节奏
- ❌ 不要在第 3 步复述前自作主张直接调 skill——必须有用户点头
- ❌ 不要试图在 router 里收集所有细节（比例、调色板、字体、模型）——下游 skill 有自己的问询流程

## 一个具体例子

用户说："帮我给这篇文章 ~/Notes/RAG-入门.md 配点图"

执行：
1. AskUserQuestion 第 1 题（7 选 1），用户选"文章正文配图（多张穿插）"
2. 第 2 步：用户已经给了文件路径，跳过
3. 复述："理解的需求：类型=文章正文配图；内容=~/Notes/RAG-入门.md；下游 skill=baoyu-article-illustrator。确认吗？" → 用户点确认
4. 调用 `Skill(skill="baoyu-article-illustrator", args="内容来源：~/Notes/RAG-入门.md\n关键约束：（无）\n其他备注：（无）")`
5. baoyu-article-illustrator 接管，开始问风格/调色板/张数
