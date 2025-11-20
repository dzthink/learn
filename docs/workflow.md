# AI 驱动的内容创作工作流设计（Draft → 发布）

版本：v1.0（面向：个人创作者 / 工程实现）

---

## 0. 总体目标

构建一条从 **碎片化输入 → AI 整理 → 文章/文集 → 衍生内容（视频/播客/社交）→ 多平台发布** 的工作流，满足：

- **手机友好**：所有关键操作可在手机完成  
- **高度依赖 AI，减少机械劳动**  
- **内容统一沉淀在 GitHub 仓库**  
- **可扩展，未来可以接更多平台 / 自动发布系统**

核心工具选择：

- **输入侧**：Apple Notes / ChatGPT（手机端）  
- **内容中枢**：ChatGPT（Draft/Idea/Article/Derivatives）  
- **内容仓库**：GitHub（Markdown + frontmatter）  
- **后续自动化**：Python/FastAPI + GitHub Actions（后续版本）

---

## 1. 角色与职责

### 1.1 你（创作者）

- 利用碎片时间记录想法、学习笔记、问题（Apple Notes / ChatGPT）
- 定期在 ChatGPT 中触发：
  - 草稿整理（Draft）
  - 大纲生成（Idea）
  - 文章生成（Article）
  - 衍生内容生成（Derivatives）
- 在手机上做轻量审核 & 逻辑修正
- 将最终版本保存/同步到 GitHub 内容仓库

### 1.2 ChatGPT（核心智能协作者）

- 清洗和结构化你的碎片内容 → Draft  
- 聚合并抽象主题 → Idea（大纲）  
- 生成完整文章 → Article  
- 生成衍生内容（视频脚本、播客脚本、公众号摘要等） → Derivatives  
- 按你的仓库规范输出 Markdown + frontmatter  

### 1.3 自动化系统（后端/脚本）

- 定期扫描 GitHub 仓库
- 根据 `status` 和 `platforms` 决定哪些内容需要发布
- 对接各平台 API，执行自动发布

> 本文档重点描述 **内容工作流**，自动发布只做接口预留。

---

## 2. 内容生命周期 & 状态机

设计一套统一的 **内容生命周期状态机**，适用于所有文章：

```text
INBOX → DRAFT → IDEA → ARTICLE_DRAFT → READY → SCHEDULED → PUBLISHED
```

简要说明：

- **INBOX**：刚记录的碎片，可能存在于 Apple Notes 或 ChatGPT 消息中
- **DRAFT**：ChatGPT 已整理成可读草稿（Draft 文件）
- **IDEA**：有了清晰文章大纲（Idea 文件）
- **ARTICLE_DRAFT**：生成了完整文章，但未最终确认
- **READY**：你已审核，内容可以对外发布
- **SCHEDULED**：已安排发布时间
- **PUBLISHED**：已在至少一个平台发布

在 GitHub 中，这个状态写在 frontmatter 里：

```yaml
status: "draft"     # 或 "ready" | "scheduled" | "published"
```

---

## 3. GitHub 内容仓库结构规范

推荐目录结构：

```bash
content/
  inbox/                    # 可选：从 Apple Notes / API 同步的原始文本
  drafts/                   # Draft 草稿（结构化但未成文）
  ideas/                    # 大纲（Idea 层）
  articles/                 # 完整文章（Article 层）
  collections/              # 文集（按主题分组）
    ai-basics/
      index.md
    ads-system/
  derivatives/              # 衍生内容
    videos/
    podcasts/
    socials/
  assets/                   # 图片、封面等
  metadata/
    index.json              # 全局索引（可选）
```

### 3.1 文件命名约定

- 草稿 (Draft)：`YYYY-MM-DD-short-keyword-draft.md`
- 大纲 (Idea)： `topic-slug-idea.md`
- 文章 (Article)：`YYYY-MM-DD-topic-slug.md`
- 衍生内容：以文章 id 为核心，例如：
  - `derivatives/videos/topic-slug.json`
  - `derivatives/podcasts/topic-slug.md`
  - `derivatives/socials/topic-slug-wechat.md`

例子：

```bash
content/drafts/2025-01-20-rlhf-reward-model-draft.md
content/ideas/rlhf-reward-model-idea.md
content/articles/2025-01-25-rlhf-reward-model.md
content/derivatives/videos/rlhf-reward-model.json
content/derivatives/socials/rlhf-reward-model-wechat.md
```

### 3.2 frontmatter 规范（Article 层）

所有 **可发布的文章**，遵循统一 frontmatter：

```markdown
---
id: "rlhf-reward-model"
title: "RLHF 奖励模型：从原理到工程实现"
date: 2025-01-25
tags: ["AI", "RLHF", "强化学习", "LLM"]
status: "ready"        # draft | ready | scheduled | published
summary: "解释 RLHF 中奖励模型的训练原理，并给出工程实现步骤和实战建议。"
platforms:
  wechat:
    enabled: true
    url: ""
  bilibili:
    enabled: true
    url: ""
  youtube:
    enabled: true
    url: ""
  website:
    enabled: true
    url: ""
---
```

---

## 4. 从 Draft 到发布：端到端流程

### 4.1 阶段 1：碎片输入（INBOX）

**目标**：你尽可能轻松地把脑中内容“扔出去”。

输入渠道：

- Apple Notes
- ChatGPT 对话（手机）
- 语音转文字 + 粘贴到 ChatGPT

示例：

> “今天学 RLHF 时，对奖励模型的训练逻辑有疑惑：为什么用 pairwise preference，而不是直接回归标量奖励？”

不需要任何结构，你只要说出来即可。

---

### 4.2 阶段 2：ChatGPT 整理为 Draft（INBOX → DRAFT）

当你有一批碎片，或随手一条时，可以对 ChatGPT 说：

> “请把上面的所有内容整理成一个 Draft 草稿。”

推荐使用的系统 Prompt 大致是：

> 你是我的 **内容草稿助手**。  
> 我会不断向你发送碎片化想法和学习笔记。  
>   
> 当我说“整理为 Draft”时：  
> 1. 将最近一批内容整理成结构化草稿  
> 2. 提取关键点和问题  
> 3. 保留原始思考，不做过度扩写  
> 4. 用 Markdown 输出  
> 5. 最后给出建议文件名（draft 文件名）  
> 6. 不要输出无关解释

输出示例（ChatGPT）：

```markdown
# RLHF 奖励模型相关问题（Draft）

## 1. 当前困惑

- 奖励模型为什么采用 pairwise preference 进行训练？
- 直接回归标量奖励是否可行？
- 这种设计对采样效率、稳定性有什么影响？

## 2. 学习笔记

- RLHF 流程概览：
  - SFT 模型
  - 奖励模型
  - Policy 优化（PPO/DPO 等）
- 奖励模型训练步骤（粗略）：
  1. 从 SFT 模型抽样多个回答
  2. 人类对回答进行排序
  3. 用排序结果训练奖励模型（更偏好高分输出）

## 3. 后续想弄清楚的方向（To Explore）

- 奖励模型目标函数的形式
- Pairwise 与 Listwise 的差异
- 对实际工程选型的影响

---

建议文件名：`2025-01-20-rlhf-reward-model-draft.md`
```

你只需 **复制这段内容 → 保存到 GitHub `content/drafts/`**。

---

### 4.3 阶段 3：从 Draft 到 Idea（DRAFT → IDEA）

当你决定要把某个话题写成文章时，对 ChatGPT 说：

> “请根据这份 Draft 生成一份文章大纲（Idea）。”

ChatGPT 任务：

- 将 Draft 内容拆解为合理结构
- 补充必要的背景、动机、案例位置
- 标注你需要补充个人经验的点

Idea 输出示例：

```markdown
# RLHF 奖励模型文章大纲（Idea）

## 1. 引言
- RLHF 在对齐大模型中的地位
- 常见误解：RLHF = PPO？

## 2. 奖励模型的角色
- 在 RLHF 流程中的位置图（Mermaid）
- 与 SFT 模型、Policy 模型的关系

## 3. 为什么用偏好建模（Pairwise Preference）
- 人类打分的噪声与主观性
- 排序信号 vs 绝对分数
- Pairwise 损失的直观理解

## 4. 奖励模型训练过程
- 数据采样 & 标注
- 模型结构（可类比分类/回归）
- 损失函数简要说明

## 5. 工程视角：设计权衡
- 采样效率
- 可扩展性
- 模型复杂度 vs 标注成本
- （作者补充：你在广告系统中见过的类似方案）

## 6. 总结与延伸阅读
- 关键要点回顾
- 推荐论文/博客
```

保存为：`content/ideas/rlhf-reward-model-idea.md`

---

### 4.4 阶段 4：生成文章（IDEA → ARTICLE_DRAFT）

当大纲确认后，对 ChatGPT 说：

> “请根据这份 Idea 生成完整文章，按我的写作风格，输出 Markdown + frontmatter。”

ChatGPT 输出示例（截取）：

```markdown
---
id: "rlhf-reward-model"
title: "RLHF 奖励模型：从直觉到工程实现"
date: 2025-01-25
tags: ["AI", "RLHF", "奖励模型", "LLM"]
status: "draft"
summary: "本文从直觉和工程视角解释 RLHF 中奖励模型的作用与训练方式，并讨论其在实际系统中的设计权衡。"
platforms:
  wechat:
    enabled: true
    url: ""
  bilibili:
    enabled: true
    url: ""
  youtube:
    enabled: true
    url: ""
  website:
    enabled: true
    url: ""
---

# 引言

（正文...）
```

此时你只需：

- 在 ChatGPT 中做少量修改
- 拷贝到仓库 `content/articles/2025-01-25-rlhf-reward-model.md`
- 可选：通过 git 提交

---

### 4.5 阶段 5：生成衍生内容（ARTICLE_DRAFT → Derivatives）

当文章基本完成，对 ChatGPT 说：

> “请基于这篇文章生成视频脚本、播客稿、公众号摘要、B站/YouTube 介绍文案。”

ChatGPT 生成统一结构，例如：

```markdown
# Derivatives for: rlhf-reward-model

## 1. B站视频脚本（10 段）

- 场景1：开场，自我介绍 + RLHF 问题抛出
- 场景2：RLHF 整体流程示意
- ...

## 2. YouTube 描述 & SEO 标签
- 描述（英文）
- tags: ["RLHF", "Reward Model", "Reinforcement Learning", "LLM Alignment"]

## 3. 播客脚本（10 分钟）
...

## 4. 公众号摘要（约 150 字）
...

## 5. 封面图 Prompt
...
```

你可以：

- 保存为 `content/derivatives/videos/rlhf-reward-model.json/md`  
- 将不同平台对应内容拆成单独文件（方便发布脚本调用）

---

### 4.6 阶段 6：审核 & 标记为 READY（ARTICLE_DRAFT → READY）

你在手机/电脑上，对文章做最终 review，完成后：

- 将 frontmatter 中 `status` 改为 `"ready"`：

```yaml
status: "ready"
```

这一动作相当于 **“加入发布候选池”**。

---

### 4.7 阶段 7：排期 & 发布（READY → SCHEDULED → PUBLISHED）

未来自动化发布系统会做：

- 扫描 `status = "ready"` 的文章
- 选择发布时间（手动 / 自动）
- 调用对应平台 API 进行发布：
  - 使用 `derivatives/socials/xxx-wechat.md` 作为公众号内容
  - 使用 `videos/xxx.json` 作为视频脚本依据（后期对接视频生成/上传）
- 发布成功后：
  - 更新 frontmatter：
    - `status: "published"`
    - 填写各平台 `url`

---

## 5. 你的日常操作清单（手机视角）

### 每天（碎片时间）

- 在 Apple Notes 或 ChatGPT 里记录任意想法：
  - 学到一个新概念
  - 对某技术的困惑
  - 一个小例子或比喻
- 偶尔对 ChatGPT 说一句：
  - “把今天的内容整理成一个 Draft。”

### 每周 1 次（内容推进日）

1. 从 Draft 中选 1–2 个话题 → 生成 Idea  
2. 确认一个 Idea → 让 ChatGPT 输出完整 Article  
3. 做轻量修改 → 存到 GitHub `content/articles/`  
4. 让 ChatGPT 生成衍生内容 → 存到 `derivatives/`  
5. 将文章 `status` 改为 `"ready"`

### 每月 1 次（文集/专题整理）

- 将多个相关文章组合成文集 `collections/xxx/`  
- 生成文集 index（可以让 ChatGPT 生成目录说明）

---

## 6. 后续扩展接口预留

为了未来方便接自动发布系统，在当前工作流中约定：

- 每篇文章都有唯一 `id`  
- 衍生内容文件命名和 `id` 对齐  
- `status` 与 `platforms` 字段统一  
- 衍生内容结构尽量固定（方便脚本解析）  

未来可以逐步加上：

- `views` / `likes` 等统计字段  
- 自动从平台抓取数据回填 `metadata/index.json`  
- 基于数据的选题 & 更新策略
