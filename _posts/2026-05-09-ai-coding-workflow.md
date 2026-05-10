---
layout: post
title: 在 Claude Code 里做长期项目的一套工作流
description: 把 AI Coding 从“会写”推进到“能长期协作、可回溯、可验收”的一套工作流
date: 2026-05-09 09:00:00 +0800
hero: /assets/images/hero.png
permalink: /posts/ai-coding-workflow/
tags:
  - AI Coding
  - Claude Code
  - Workflow
---

<section class="article-brief" aria-label="文章摘要">
  <div>
    <span>一句话版</span>
    <p>这篇文章不是讲“如何让 AI 更快写代码”，而是讲“如何让 AI 在中长期项目里稳定、可交接、可验收地持续工作”。</p>
  </div>
  <div>
    <span>适合谁</span>
    <p>单人或 2-3 人紧密小团队，项目周期长，AI 参与深，且愿意先把规则、文档和验证链路搭起来。</p>
  </div>
  <div>
    <span>不适合谁</span>
    <p>一次性脚本、小工具、快速原型，或者完全不想维护项目级规则的人。</p>
  </div>
</section>

<nav class="reading-map" aria-label="一分钟阅读地图">
  <strong>一分钟阅读地图</strong>
  <a href="#最小起步版本如果你只想先试一试"><span>3 分钟</span>最小起步版本</a>
  <a href="#六层资产先看整体"><span>15 分钟</span>六层资产和两段流程</a>
  <a href="#踩过的坑"><span>对照排坑</span>踩过的坑与局限</a>
  <a href="#参考资料"><span>查事实</span>官方文档和社区实践</a>
</nav>

---

## 起因

用 Claude Code 做单个功能、修几行 bug、写个脚本的时候，AI 很强。**但一旦项目跨进"两周以上、多模块协作、多轮对话积累上下文"的中长期阶段**，就开始撞墙：

- **AI 走偏**：写着写着给简单函数加抽象层；让它实现 A，回来发现它顺手改了 B、重写了 C
- **上下文压缩的断档**：压缩后下一轮对话"忘了"前面的设计意图，凭印象重新拼凑
- **几个月后看不懂**：回头看代码，完全不记得当时为什么这么设计
- **新人接手门槛高**：团队新人（或三个月后的自己）只能对着代码反推意图
- **经验不复用**：踩过的坑、总结的套路，换个项目又从零开始

这些痛点在**短对话、一次性任务**里几乎感觉不到——AI 足够快、结果看上去就对。**它们只有在项目跑过一段时间、复杂度上来之后才会显现**。这也是为什么很多人说"AI coding 挺好用的"而另一些人说"AI 长期做项目还是不行"——两边都没说错，只是在讨论不同阶段的事。

问题的根源不是模型能力，是**组织工作的方式不对**。人在长期项目里能走下去，靠的是设计文档、代码评审、CI、团队约定这一整套**外部约束**；AI 放进同一个项目却让它裸跑，肯定要出问题。

过去一两年摸出一套自己的打法，写出来一方面梳理思路，一方面看看社区有没有人做类似的事。受 [Obra 的 superpowers](https://github.com/obra/superpowers) 启发很多（brainstorm → plan → TDD → review 那套纪律），但因为我的项目里**人类写的架构文档本身也是一等事实源**，所以扩展了不少。

---

## 整套工作流分两段

很多人讲 AI coding 工作流时只讲"接到需求后怎么开发"这一段。但我发现**项目初始化这一次**做不做得扎实，直接决定后面所有开发是顺还是糟。所以我把工作流切成两段：

- **项目初始化**（只做一次，半天到一天）：把不变的规则 / 子 Agent / Skill / Hook 配齐
- **需求开发**（每个需求跑一轮）：调研 → 分析 → 改文档 → 改代码

下面分开讲。

---

## 最小起步版本（如果你只想先试一试）

六层资产和两段工作流看起来很复杂。完全不用一上来全装——**真正不可省的核心只有两条**：

<div class="callout callout--important">
<p class="callout__eyebrow">硬核两条</p>

<ol>
<li><strong>改代码前先改文档</strong>：任何非琐碎变更，先写一份 proposal（说清楚做什么 + 怎么验收 + 涉及哪些模块），再让 AI 动代码。</li>
<li><strong>测试先讲清楚</strong>：proposal 里列好验收标准和测试策略。编码时同步写测试。没测过的代码不归档。</li>
</ol>
</div>

只做这两条，已经能挡住长期项目里 80% 的走偏和返工。

其它一切（子 Agent 分层、Hook 拦截、定时任务、Agent 私有记忆……）都是在这个核心上的**渐进扩展**——项目跑起来后遇到具体痛点再加，不是一上来就配齐。**这是读本文的正确姿势**：六层骨架是终态的全景地图，不是每个人一开始都要装满。

---

## 六层资产：先看整体

在展开两段流程之前，先给一张全景图。项目里围绕 AI 协作的所有资产可以分成六层——后面讲的东西都能对应到这张表的某一层：

| 层 | 内容 | 更新频率 | 位置 |
|---|---|---|---|
| L1 静态规则 | 技术栈 / 编码规范 / Git 协作 / 项目结构 | 初始化后基本不动 | `.claude/rules/` + `CLAUDE.md` / `AGENTS.md` |
| L2 子 Agent | 需求 / 研究 / 编码 / 测试 / 评审 / 文档等角色 | 偶尔扩展 | `.claude/agents/` |
| L3 Skill | 固定流程封装 + 社区 Skill 复用 | 按需新增 | `.claude/skills/` + 装的 plugin |
| L4 Hook | 工具调用拦截（事实源保护、静态检查） | 基本固定 | `.claude/settings.json` / `hooks/` |
| L5 动态经验 | 踩坑沉淀 + Agent 私有记忆 + Auto Memory | 持续增长 | `.claude/agent-memory/`（Agent 私有）+ 项目 `memory/`（团队共享）+ `~/.claude/projects/.../memory/`（Auto） |
| L6 定时任务 | 巡检 / 周报 / harness 自审 | 装一次用很久 | `~/.claude/scheduled-tasks/` |

两段流程里，**项目初始化**负责把 L1-L4 装齐；**需求开发**在这套骨架上跑每一轮变更，同时让 L5 随时间积累；L6 定时任务是可选的后装项，到"让 harness 自己跑起来"那节会单独讲。

---

## 第一段：项目初始化

### 1. 告诉 Claude Code 项目的静态规则

新项目的第一件事，我会和 Claude Code 对话式地把几类**几乎不会变的规则**说清楚：

- **技术栈**：用什么语言、框架、数据库、部署环境；版本要求；不允许引入什么
- **编码规范**：命名习惯、缩进、错误处理风格、日志规范、注释原则
- **Git 协作规范**：分支命名、提交信息格式、PR 模板
- **项目结构约定**：目录怎么切、模块怎么划边界
- **协作规范**：任务怎么拆、评审走什么流程、谁负责什么

要点是**尽量简洁可验证**——能写成"命令格式是 `feat(scope): 中文描述`"就不要写成"提交信息要有意义"；能写成"用 pnpm 不用 npm"就不要写成"包管理要规范"。官方文档也明确说过：[规则越具体、Claude 越能可靠遵守](https://code.claude.com/docs/en/memory)；含糊的描述会被它忽略。

写完让 Claude Code 整理成 markdown 落到项目里。这就是 L1 静态规则层，每份一页以内——Claude 加载它们要消耗 token，长了反而执行效果变差（[官方建议每份 CLAUDE.md 控制在 200 行以内](https://code.claude.com/docs/en/memory)）。

**放哪里很有讲究**，这块后面有专门一节讲清楚 Claude Code 的记忆机制。

### 2. 定义可能用到的子 Agent

接着让 Claude Code 按项目特点**自己提议一套子 Agent**。我会告诉它我的常见工作模式，让它建议要哪几类职责。通常围绕这些角色展开（命名随意）：

- **需求分析**：把口头需求转成结构化的需求分析文档
- **技术调研**：技术调研，比较方案，给结论
- **编码**：按设计文档写实现
- **测试**：写单元/集成测试，跑测试，报失败
- **评审**：对着设计文档审改动
- **文档**：维护架构文档、特性文档、README

**不是每个项目都要这么多**——纯前端项目可能多一个 UI 设计的角色，后端重的项目可能多一个数据库 schema 的角色。关键是让子 Agent 按**工作阶段**划分（需求/研究/编码/测试/评审/文档），而不是按**技术领域**划分（前端/后端/数据库）。前者可以跨项目复用，后者只对当前项目有效。

**为什么要分子 Agent**：
- **可并行**：调研和需求分析可以同时跑
- **上下文干净**：每个 Agent 只加载自己需要的规则，不用把整个项目的 CLAUDE.md 全塞进去
- **职责边界拦错**：让"编码"角色做调研，它会凑一份看起来合理但可能错的答案；让独立的"调研"角色做，结果稳得多
- **并行到 worktree 层**：互不依赖的需求直接拆成独立 worktree / 会话，review 用 fresh context，只看代码和设计，不吃实现会话的推理历史

### 3. 先去社区看看，别什么都自己写

这块我一开始踩过坑——什么 Skill 都自己从头写，后来才发现社区已经有很多好东西。常用的几个我实际在装的：

| Skill / Plugin | 作用 | 适合谁 |
|---|---|---|
| [`superpowers`](https://github.com/obra/superpowers) | 一套完整的 brainstorm → plan → TDD → review 纪律工作流；会强制让 AI 先想清楚再动手 | 做复杂功能、不想被 AI 乱改的人 |
| [`skill-creator`](https://github.com/anthropics/skills/tree/main/skills/skill-creator) | Anthropic 官方的 Skill 开发工具链；从写 SKILL.md 到测试到优化全包 | 想开发自己的自定义 Skill |
| [`ui-ux-pro-max`](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill) | UI/UX 设计辅助：根据产品类型生成完整设计系统（配色 / 字体 / 布局 / 反模式清单）；支持多种前端栈 | 做前端 / 产品界面 |

**实际使用策略**：我一般**装了整个 plugin，但只用其中一两个 Skill**。比如装了 superpowers，我主要用它的 `brainstorm` 和 `plan`；TDD 强制那部分我跟我的工作流不完全对味，就不主动触发。

**找 Skill 的渠道**：
- Claude Code 官方 marketplace（`/plugin marketplace` 命令里面有）
- GitHub 搜 `claude-code-skill` / `claude-skill` tag
- [claudepluginhub.com](https://www.claudepluginhub.com/) / [playbooks.com](https://playbooks.com/) 这类聚合站

装之前一定看一眼 SKILL.md 里的 frontmatter 和 `description` 字段——Claude 就是靠 description 判断何时触发这个 Skill 的，看一眼能知道它会在哪些场景下自动激活，避免装了之后跟自己的工作流打架。

### 4. 定义常用 Skill（自己项目的工作流编排）

除了社区的，我会给项目加自己的几条编排 Skill。Skill 是"**固定流程**的封装"——一份带 frontmatter 的 markdown，告诉 AI 这个流程怎么走。具体命令名随意，重要的是封装**在这个项目里经常重复的固定动作**。常见能想到的几类：

- 创建需求分析三件套（需求分析 + 变更清单 + 任务清单）
- 按任务清单推进到归档
- 开发修复循环（定位 → 最小复现 → 写测试 → 修）
- 对着设计文档审最近的改动
- 把踩过的坑沉淀进项目经验库

**Skill 和子 Agent 的区别**：Skill 是**流程**（"这件事该怎么做"的步骤），Agent 是**角色**（"谁来做"的身份与限制）。一个 Skill 里通常会调度多个 Agent——比如"创建需求分析三件套"这个 Skill，可以先让调研角色做调研，再让需求分析角色做分析，最后让文档角色起草草案。

### 5. 定义可能需要的 Hook

Hook 是在**工具调用的瞬间**拦截的钩子。Claude Code 官方有 `PreToolUse` / `PostToolUse` / `Stop` / `SessionStart` / `InstructionsLoaded` 等多种事件。

我常挂三类：

1. **事实源保护（PreToolUse）**：AI 想改设计文档或核心业务模块时，检查是否有对应的需求分析文档在进行中。没有就拒，并提示"请先走 /spec-propose"。这是后面 spec-first 最硬的那层执行保障。
2. **静态检查（PostToolUse）**：AI 改完文件后自动跑 linter / typecheck，失败了回传错误让它自己修。
3. **执行边界（Policy）**：把可写目录、危险命令、网络访问白名单和 secret 处理写清楚。常规编辑可以自动放行，越界操作要显式确认。

Hook 是最后一道闸——前面 Skill / Agent / 规则都是"约定"，Hook 是"技术拦截"，AI 想绕开规则直接改文件，在工具调用层就被 deny。

### 给 AI 一套可验证的执行环境

我现在越来越重视把常用动作收敛成固定命令：`make dev`、`make test`、`make lint`、`make tail-logs`。最好再有一个统一的日志入口，把服务端错误、构建输出和关键告警集中起来。这样 AI 不用在多个终端里猜状态，验证也更快。

这件事看起来像工程卫生，但对 agent 来说很关键：命令越少、日志越统一，它越容易自己跑完一轮并修回去。

### 初始化阶段做完的样子

这几件事做完，项目大致会长出下面这种骨架。**具体目录名、具体 Agent 名、具体 Skill 名都不重要**——重要的是每层的**职责边界**要清楚：

```text
your-project/
├── CLAUDE.md                   # 项目地图：Claude Code 新会话自动加载
├── .claude/
│   ├── settings.local.json     # 权限 + 本地 Hooks
│   ├── rules/                  # L1 静态规则（按主题分文件）
│   │   └── *.md                # 技术栈 / 编码规范 / 项目结构 ...
│   ├── agents/                 # L2 子 Agent（按工作阶段划分：需求/研究/编码/测试/评审/文档 ...）
│   │   └── *.md
│   ├── agent-memory/           # L5 的一部分：每个 Agent 的私有记忆
│   │   ├── MEMORY.md           # 记忆索引
│   │   └── <agent-name>/       # 对应 agent 的积累
│   ├── skills/                 # L3 Skill（自己项目的编排 + 从社区装的）
│   │   └── <skill-name>/
│   └── hooks/                  # L4 Hook 脚本（配合 settings.local.json 的 hooks 配置）
├── docs/                       # 类别 A：项目资产文档
└── specs/                      # 类别 B：开发过程文档（每次需求的三件套）

# 用户级别（~/.claude/ 下，不进项目 git）
~/.claude/
└── scheduled-tasks/            # L6 定时任务（下面专门讲）
```

几个值得单独拎出来说的思想：

- **Harness 总纲独立成文**：我的做法是把"这个项目的 AI 协作哲学"抽一份独立文档放在 `.claude/` 下（名字随便，比如 `AGENT_HARNESS.md`），在 `CLAUDE.md` 里用 `@` 引进来。这样 `CLAUDE.md` 顶层保持极简（一页以内的地图），具体 harness 指令在独立文件里演化。
- **养一个 meta agent 审查 harness 自身**：可以定义一个职责是"检查 harness 是否健康"的子 Agent——检查 rules 有没有过期、agent 之间职责有没有漂移、memory 里有没有过多冗余。这个 agent 定期跑（见下面的定时任务），帮你保持 harness 不烂。
- **Agent 私有记忆 ≠ 项目共享记忆**：Claude Code v2 的[子 Agent 持久记忆](https://code.claude.com/docs/en/sub-agents#enable-persistent-memory)允许每个 Agent 有自己的记忆空间。启用后 Claude 会自动在 subagent 的系统提示里注入记忆读写指引、加载 `MEMORY.md` 前 200 行、并自动开启 Read/Write/Edit 工具让它管理自己的记忆。位置按 scope 区分：
  - `user` → `~/.claude/agent-memory/<agent-name>/`（跨所有项目）
  - `project` → `.claude/agent-memory/<agent-name>/`（官方推荐默认，进 git 共享）
  - `local` → `.claude/agent-memory-local/<agent-name>/`（本项目私有，不进 git）
- **定时任务在 `~/.claude/` 下**：这是 Claude Code 官方约定的用户级位置，不进项目 git，但可以对项目发挥作用。下面专门讲。

---

## 第二段：需求开发（每次跑一轮）

### 项目里的两类文档

初始化做完，每次新需求来了，就进入这一段。先说明：在我这里"文档先行"里的**文档**，和 superpowers 里的 plan 不完全一样。我区分两类：

#### 类别 A：项目资产文档（给人看、给未来看）

这些是项目**真正的知识资产**，和代码同等重要：

- **整体架构文档**：系统架构图、关键时序图、跨模块的数据流、技术选型决策
- **整体特性说明文档**：这个系统对外提供什么能力、边界在哪
- **子模块架构文档**：每个主要模块内部的设计、类图、模块内的交互
- **子模块特性说明文档**：每个模块对外暴露什么接口 / 契约
- **README**：项目入口说明

这些不是"配合 AI 写的文档"。这是**你的项目本来就应该有的设计文档**，只不过我让 Claude Code 帮我维护它们、并且要求它改代码前先动它们。

#### 类别 B：开发过程文档（给 AI 和交接用）

这是 AI 协作流程自己产生的文档：

- **需求分析文档**：每次 `/spec-propose` 的产出，含需求背景、验收标准、边界约束、涉及模块
- **变更记录**：这次变更改了哪些设计文档、哪些代码文件、为什么改
- **工作区记录**：最近在做什么，进行到哪一步，下一步要干嘛——**为跨会话 / 跨 AI 交接准备**
- **踩坑经验库**：持续沉淀的 memory 条目

这两类合起来才是项目的完整事实源。**类别 A 是长期、稳定的**；**类别 B 是过程性的，有些归档、有些滚动更新**。

### 需求开发的标准流程

一个新需求进来，我一定走这 5 步，顺序不能乱：

<div class="workflow-roadmap" aria-label="需求开发的五步流程">
  <div class="workflow-step">
    <span>1</span>
    <strong>需求调研</strong>
    <p>子 Agent / brainstorm</p>
  </div>
  <div class="workflow-step">
    <span>2</span>
    <strong>需求分析</strong>
    <p>生成三件套，写清验收标准、边界和测试</p>
  </div>
  <div class="workflow-step workflow-step--doc">
    <span>3</span>
    <strong>改设计文档</strong>
    <p>子模块 → 整体 → README</p>
  </div>
  <div class="workflow-step workflow-step--code">
    <span>4</span>
    <strong>改代码</strong>
    <p>按任务清单推进，编码同步写测试</p>
  </div>
  <div class="workflow-step">
    <span>5</span>
    <strong>归档 &amp; 记录</strong>
    <p>跑测试，沉淀变更记录、工作区记录和 memory</p>
  </div>
  <p class="workflow-loop">下次需求回到第 1 步</p>
</div>

**关键：第 3 步和第 4 步不能倒过来**。反过来（先改代码再补文档）在 AI coding 场景下几乎必然漏：AI 改完代码会本能觉得任务完成，补文档变扣分题；过一段时间代码再改，文档对不上就烂了；下次 AI 读文档得到错的前提继续改，恶性循环。

先改文档再改代码强迫我把设计想透一轮，而且代码改下去时文档已经是最新的参考，AI 不会被过时文档误导。

### 先做 scout / spike，再进正式 proposal

对陌生模块、陌生依赖，或者边界本来就不清楚的需求，我会先让调研 Agent 做一次 scout：只找入口、风险点、未知约束，最多做一个很小的 PoC，不直接开大实现。等把“不确定清单”收敛掉，再进入正式 proposal。

这一步的价值是把"我其实还没搞懂"尽早暴露出来，不让正式设计建立在模糊假设上。

### 哪些改动可以跳过这套流程

不是每个改动都值得走完整的 5 步。我的经验边界：

**✅ 必须走流程**：
- 新功能 / 新模块 / 新接口
- 修改业务逻辑（会影响系统对外行为的）
- 架构层变动（拆分模块、引入新依赖、调整数据流）
- 安全 / 权限 / 数据模型相关

**⚠️ 可以跳过**：
- typo、注释调整、格式化
- 单个测试补充（不改实现）
- 明确的 hot-fix（回滚或最小修补，但必须在归档时补登记）
- 依赖版本 bump（除非是 major 或含 breaking change）

**判断准则**：这个改动**会影响别人对系统的理解吗**？会就走流程，不会就跳。不确定时走流程更安全——代价只是多花 10 分钟写 proposal，但漏了一次会制造后续几天的混乱。

### 测试左移的具体做法

这点很重要，单独讲。

**第 2 步"需求分析"阶段**就要把测试想清楚，具体内容让 AI 写进需求分析文档：

- **验收标准**：满足什么条件算做完（用 EARS 句式或 Given-When-Then 都行，能验证就行）
- **边界约束**：输入的极端情况、空值、异常、并发
- **测试策略**：哪些测试点能用**单元测试**覆盖，哪些需要**集成测试**，哪些必须**端到端**

**第 4 步编码**时，**能用单元测试或集成测试覆盖的，就在编码时同步覆盖**。AI 写一段实现，同时写一组测试——这不是严格 TDD（不要求先红后绿），但保证了测试和实现是**在同一个上下文下产出**的，比事后补测试质量高得多。

**第 5 步归档前**，一定跑一次测试。

**端到端测试**分三种处理：

1. **浏览器里的 E2E**：装 [Claude in Chrome 扩展](https://docs.anthropic.com/en/docs/claude-code/chrome)（beta），然后 `claude --chrome` 启动，或在会话里 `/chrome`。Claude Code 会通过 MCP 通道接管一个 Chrome 窗口——可以点页面、填表单、读 console、抓截屏。写好要测什么、期望什么，它跑完一轮给你报结果。
2. **后端 API 的 E2E**：直接让 Claude curl 或写一个集成测试脚本自己跑。Claude Code 有 Bash 工具，这类任务不需要额外插件。
3. **复杂的人工 E2E**：AI 输出一份"怎么测"的测试脚本或步骤清单，我人肉走一遍。测完把结果（通过 / 失败哪步）喂回去。

这套下来，**没测过的代码不归档**是硬约束。归档时往变更记录里写一句"测试覆盖情况 + 端到端通过"，以后回溯能看到。

### 把需求分析做成活文档

我现在更倾向把需求分析写成一个能持续维护的 living spec / ExecPlan，而不是一次性 proposal。最少保留这几块：
- `Progress`：当前做到哪一步
- `Surprises & Discoveries`：实现中发现了什么新事实
- `Decision Log`：关键决策和原因
- `Outcomes & Retrospective`：最终验证结果和复盘
- 每个里程碑对应的命令、输出和验收信号

这样“工作区记录 / 需求分析 / 变更记录”三者的边界会更清楚：需求分析管计划和决策，工作区记录管当前进度，变更记录管归档结果。

---

## 用好 Claude Code 原生记忆

Claude Code 有**两套并行的记忆系统**（[官方文档](https://code.claude.com/docs/en/memory)）：

### 1. CLAUDE.md（你写的指令）

新会话启动时自动加载到上下文。有完整的层级结构（从高到低）：

| 层级 | 位置 | 用途 | 进 git？ |
|---|---|---|---|
| 企业级 | `/Library/Application Support/ClaudeCode/CLAUDE.md` (macOS) 等 | 组织统一下发，IT/DevOps 管 | ❌ |
| 项目级 | `./CLAUDE.md` 或 `./.claude/CLAUDE.md` | 团队共享的项目规则 | ✅ |
| 用户级 | `~/.claude/CLAUDE.md` | 你自己的偏好，跨所有项目 | ❌ |
| 项目本地 | `./CLAUDE.local.md` | 你在当前项目的私人偏好 | ❌（加 `.gitignore`） |

**几个好用但容易漏的细节**：

- **`.claude/rules/*.md` 模块化规则**：不用把所有规则堆到一个 `CLAUDE.md`。分成 `testing.md`、`api-design.md`、`security.md` 这样的小文件放到 `.claude/rules/`，每份一主题。更牛的是支持 frontmatter 的 `paths` 字段——规则可以**只在 Claude 读匹配路径的文件时才加载**：

  ```yaml
  ---
  paths:
    - "src/api/**/*.ts"
  ---
  # API Development Rules
  - 所有 API endpoint 必须输入校验
  - 错误响应用统一格式
  ```

  这比全局 CLAUDE.md 省 token、规则定位准。

- **`@import` 语法**：CLAUDE.md 里可以写 `@docs/architecture/overview.md` 把别的文件引入（支持递归，最多 5 层）。这样项目的架构总览文档能直接作为 AI 上下文的一部分。

- **`/init` 命令**：新项目直接跑 `/init`，Claude 会分析代码库自动生成 CLAUDE.md 骨架。已经有 CLAUDE.md 它会给改进建议而不是覆盖。

- **`/memory` 命令**：会话中查看所有加载了哪些 CLAUDE.md / rules 文件，顺手能打开编辑。排查"Claude 不听话"时先跑这个。

### 2. Auto Memory（Claude 自己写的笔记）

Claude Code v2.1.59 起自带的功能，**默认开**。

Claude 在会话里干活时，如果判断"这条信息以后有用"，会**自己决定**写进 `~/.claude/projects/<project>/memory/` 下。结构是：

- `MEMORY.md` — 索引，会话启动时自动加载前 200 行 / 25 KB
- 若干主题文件（`debugging.md` / `api-conventions.md` 等）— 不自动加载，Claude 按需读

**最实用的一点**：当你对 Claude 说"记住以后用 pnpm 不用 npm"、"测试命令是 `make test-integration`"，它就会自动存到 Auto Memory。不用手动开文件。

我自己的做法：
- **重要的、团队共享的规则**：放项目级 `CLAUDE.md` / `.claude/rules/`，进 git
- **个人习惯 / 当前项目临时的东西**：放 `CLAUDE.local.md`，不进 git
- **Claude 自己发现的模式**：让它用 Auto Memory 自己管，偶尔 `/memory` 翻翻它记了什么，错的删掉

**Auto Memory 的局限**：机器本地存储，换机器 / 换 worktree 不同步；Codex 看不见。真正重要的项目知识还是得落到 git 里的 CLAUDE.md / rules / 设计文档。

### 跨 AI 工具迁移怎么办

好消息：**项目级规则文件这层已经开始标准化了**。不用再手写两份。

[**AGENTS.md**](https://agents.md/) 是 OpenAI 发起、现由 Linux Foundation 的 Agentic AI Foundation 治理的跨工具开放标准。目前读 AGENTS.md 的工具覆盖了主流的几乎所有家：**Codex CLI、Cursor、Windsurf、GitHub Copilot、Devin、Amp** 等。

Claude Code 的策略也很合理：

- **原生只读 CLAUDE.md**，但 `/init` 命令会**自动读取** `AGENTS.md` / `.cursorrules` / `.windsurfrules`，把相关部分合并进生成的 CLAUDE.md
- 你也可以手动在 `CLAUDE.md` 里写一句 `@AGENTS.md`，把 AGENTS.md 内容显式引进来
- 或者直接符号链接 `ln -s AGENTS.md CLAUDE.md`（Windows 上要 Administrator 权限）

所以**实际操作**是：**把团队共享的项目规则主写在 `AGENTS.md` 里**——Codex / Cursor / Copilot / 等直接读，Claude Code 通过 `@import` 也读到。一处维护，各处生效。Claude-specific 的指令追加在 CLAUDE.md 里即可（[官方推荐的模式](https://code.claude.com/docs/en/memory)）。

**但以下东西目前还是跨不过去**：

- **个人偏好 / 用户级记忆**：每家有自己的 `~/.claude/CLAUDE.md`、Codex 的 `~/.codex/` 配置、Cursor 的 User Rules 等
- **Auto Memory / 子 Agent 私有记忆**：这类工具自动积累的笔记是工具内部状态，没有跨工具导入机制
- **Skill / 子 Agent / Hook 的具体 schema**：格式各家不一样，Claude Code 的 Skill 放到 Codex 上不能直接用。社区有一些桥接工具在尝试解决：[ClaudeKit migrate](https://docs.claudekit.cc/docs/cli/migrate) 能把 Claude 格式转到其它家、[ai-config-sync](https://github.com/slash9494/ai-config-sync-manager) 做 Claude Code ↔ Codex 双向 diff+apply 同步——都能用但还没成熟到可以盲装。

结论：**项目规则用 AGENTS.md 做单一事实源**，几乎零成本跨工具；**工具内部状态的损失**是切换工具时不得不吃的成本，目前没有好办法。

---

## 长上下文处理

Claude Code 会话跑久了上下文会满。我的策略：

- **一般开发需求**：上下文满了就**新开会话**。结束前让 Claude 把当前进度写进工作区记录（或者用 Stop Hook 自动触发），下次新会话先读一遍工作区记录继续。这比 `/compact` 干净，丢的信息也少。
- **长程自主任务**（比如让它跑一晚做一次大重构）：让它自己 `/compact`。因为我不在旁边，没法做交接。接受一定的信息损失。

工作区记录里写的是**当前进行态**：
- 现在做的需求是哪个（指向需求分析文档）
- 做到哪一步了
- 下一步要干什么
- 最近发现的坑、临时决策

**这是跨会话、跨 AI 交接的关键**。每次新会话 AI 先读这份记录就能无缝接上，避免了每次都要重新"介绍一下进度"。

### 再补一份代码地图

规则文件回答的是"你该怎么做"，但我还想再补一份很薄的 context map，专门告诉 AI "代码从哪儿进、命令从哪儿跑、日志去哪儿看"。可以单独维护一个 `docs/code-map.md` 或 `.claude/context-map.md`，只写：
- 主入口
- 核心模块
- 常用命令
- 日志位置
- 测试入口
- 新会话先读的 3 个文件

这类索引可以由定时任务或轻量脚本半自动更新。它不是架构文档，也不是 CLAUDE.md，而是 AI 的导航图。

---

## 让 harness 自己跑起来：定时任务（L6）

前面 5 层都是**你主动发起对话**时才生效。但有一类工作是你在写代码时不会主动想起来的，又确实该做——**巡检**类的事情。这就是定时任务的位置。

Claude Code 官方支持三种调度（[定时任务文档](https://code.claude.com/docs/en/desktop-scheduled-tasks) + [Routines 文档](https://docs.anthropic.com/en/docs/claude-code/web-scheduled-tasks)）：

| 方式 | 跑在哪 | 需要电脑开机 | 文件访问 | 触发方式 | 适合什么 |
|---|---|---|---|---|---|
| **Desktop 本地定时任务** | 本机 | ✅ | 完整本机文件 + 本机工具 | 时间（最短 1 分钟间隔） | 需要访问本机私有文件 / 工具的日常巡检 |
| **Routines（云端）** | Anthropic 云 VM（4 vCPU / 16GB） | ❌ | 克隆你的 repo，看不到本机未提交文件 | 时间（最短 1 小时）/ HTTP POST / GitHub 事件 | 能在 git 仓库里闭环的作业（PR 自动评审、发布前检查） |
| **`/loop` 会话内** | 当前会话 | ✅ | 跟随会话 | 固定间隔（按指定周期循环触发） | 一次性临时 polling（盯 CI / 轮询状态） |

我用的是 **Desktop 本地定时任务**——因为巡检内容几乎都要读本机里跑着的服务和配置。**如果你的任务只需要 repo 内容**（比如"每天审查新增 commits 的编码风格"），Routines 更合适：电脑关机也能跑、还能被 GitHub 事件触发。

### 创建方式

两种：

1. **Desktop app 的 Routines 面板点击创建**：填名字、prompt、folder、schedule
2. **直接在会话里用自然语言**："设一个每周一早上 10 点的定时任务，让 docs-code-sync 检查文档和代码是否同步"——Claude 自己创

创建后会落到 `~/.claude/scheduled-tasks/<task-name>/SKILL.md`——本质就是一份带 frontmatter 的 Skill。每次到点 Claude Code 会**开一个全新会话**跑这个 Skill。

### 我会用定时任务跑什么

与其列我的具体任务名，不如说说**哪类工作适合让定时任务去做**——这样你可以按自己的项目设计：

- **巡检类**：每天 / 每周扫一遍特定维度的问题。典型的比如"最近的 commits 有没有违反编码规范"、"安全扫描有没有新发现"、"依赖版本有没有过期"。你不会主动想起来做，但定期做一下能拦住很多慢慢烂掉的东西。
- **同步类**：让 AI 定期核对两个应该保持一致的地方是否真的一致。典型的是"代码和文档有没有脱节"、"spec 归档的描述和实际改动有没有对得上"。这类检查人工做烦、AI 做刚好。
- **Harness 自审类**：让一个 meta agent 定期检查 harness 自身——rules 有没有冲突、agent 职责有没有漂移、memory 里有没有过期条目。不做的话 harness 会慢慢烂，没人注意到。
- **周期性汇总类**：比如每周五做一次本周回顾——整合 commits / 归档的 specs / 关闭的 issues / 测试覆盖变化，生成一份人类可读的周报。比自己凭印象回忆可靠得多。
- **监控和提醒类**：某个长跑的构建完没完 / 某个 PR 的 CI 过没过 / 某个环境的日志有没有报错。这类更适合 `/loop` 会话内做，但也能用定时任务固化下来。

关键原则：**定时任务要能自己闭环**。要么是纯报告（只读、不需要你看就能自己过），要么一次性把修改改掉（比如自动提一个 PR）。**"定时跑完输出报告但必须等你决策"这种任务价值最低**——它只是把"记得做"换成了"记得看"，没真省事。

定时任务我整体用得不多，但上面这几个装上之后，项目整体"自我保持健康"的能力明显上一个台阶。**很多琐事不是做不到，是没人记着做**；让 AI 按时做就完事。

---

## 六层资产，两层值得多说一下

开头那张六层表回顾一下——除了 L1-L4 的护栏外，真正值得多说两句的是 L5 和 L6：

**L5（动态经验）是整套工作流里复利最大的一层**。L1-L4 是护栏，不同项目的结构大同小异；L5 是"这个项目独有的知识资产"，持续增长。换项目时 L5 是唯一能完整带走的东西。L5 内部又分三路存储，来源不同：
- **Agent 私有记忆**（`.claude/agent-memory/`）：每个子 Agent 自己积累的领域经验，Claude 自动维护，跨会话可用
- **项目共享经验库**（你自己维护的 `memory/` 目录）：团队约定、踩过的坑、决策记录，进 git
- **Auto Memory**（`~/.claude/projects/.../memory/`）：Claude 根据你的纠正自动存的笔记，机器本地、不同步

**L6（定时任务）是让整套 harness 从"等你调用"变成"主动保持健康"的转折点**。用得不多，但装上之后项目"自我维护"的能力会上一个台阶。

---

## 踩过的坑

<details markdown="1">
<summary>坑 1：把项目设计文档和需求分析文档混在一起</summary>

早期我不分类别 A 和类别 B，所有文档都扔进 `docs/`。结果看项目架构的人翻出一堆废弃的需求分析文档；想找变更记录的人淹没在架构文档里；AI 分不清哪些是事实源、哪些是过程档案。

后来分开放：类别 A 在 `docs/architecture/` 和 `docs/features/`，类别 B 在 `specs/<change-id>/` 和 `memory/`。立刻清爽。

</details>

<details markdown="1">
<summary>坑 2：需求分析文档写得像设计文档</summary>

需求分析最开始越写越长，开始混入实现细节和架构描述。后来强制自己：**需求分析只管"做什么 + 什么叫做完了 + 涉及哪些模块 + 怎么测"，不管"怎么做"。**

"怎么做"属于类别 A 的架构文档——需求分析里只能 reference 到"会更新架构文档的 XX 章节"，不能把更新内容直接写在需求分析里。

分清楚之后，每份需求分析控制在 1-2 页；架构文档单独演化。

</details>

<details markdown="1">
<summary>坑 3：Hook 拦太狠</summary>

一开始事实源保护 Hook 的覆盖范围太宽，AI 连小改动都要先走需求分析。一次会话被拦好几次，很快失去耐心。

后来改成：**只保护关键路径**——架构文档、核心业务模块、配置文件。琐碎改动可以绕过。约束要严在关键处，而不是到处都严。

</details>

<details markdown="1">
<summary>坑 4：工作区记录一开始没做</summary>

早期没有这份记录，经常出现"今天做一半，明天新开会话从头问一次进度"的情况。问到后来自己都记不清，更别说让 AI 接上。

后来把工作区记录作为每次会话结束前的必做动作（可以手动维护，也可以用 Stop Hook 自动提醒），下次新会话 AI 自己先读。从此断档消失。

</details>

<details markdown="1">
<summary>坑 5：CLAUDE.md 越写越大</summary>

最早什么都往 CLAUDE.md 塞。超过一定长度后 AI 开始不遵守规则——官方文档明确说过"[过长会降低 adherence](https://code.claude.com/docs/en/memory)"。

后来拆成：
- `CLAUDE.md` 保留项目全局地图（尽量控制在 100 行以内）
- 具体规则拆到 `.claude/rules/*.md`，按主题分文件
- 路径专属规则加 frontmatter `paths` 字段，按需加载

瘦下来之后规则遵守度立刻回来。

</details>

<details markdown="1">
<summary>坑 6：Skill 全都自己写</summary>

初期所有 Skill 都自己从零起草，超耗时间。后来发现社区已经有很多成熟资产（[superpowers](https://github.com/obra/superpowers)、[Anthropic 官方 skills 仓库](https://github.com/anthropics/skills) 等），装上用就行。

教训：**先去 Claude Code marketplace 和 GitHub 找一圈**，找不到再自己写。自己写的 Skill 留给**真正项目特有的流程**。

</details>

---

## 已知的局限（这套方法论解决不了的）

诚实说几条方法论本身的局限——不是执行问题，是**它自己的盲区**。

<details markdown="1">
<summary>1. 文档和代码的“假一致”风险无法靠这套流程消除</summary>

<div class="callout callout--warning">
<p class="callout__eyebrow">警惕</p>

<p>这是<strong>所有 AI coding 方法论的通病</strong>，不光是本文这套。文档越完备，越容易让所有人（包括你自己）以为代码一定符合文档——警惕这种心理舒适感。</p>
</div>

场景：AI 改了文档说"用户模型增加 subscription_id 字段，外键关联 subscriptions 表"；AI 接着改代码，加了 subscription_id 但漏了外键约束；测试凑巧没覆盖 FK；review agent 看到文档和代码"各自自洽"，给出通过。结果代码和文档各自成立但拼起来是错的。

AI 特别擅长制造这种假一致——它记得自己在文档里写了什么，然后让代码去匹配"文档的叙述"，而不是匹配现实约束。review agent 读同一套材料时也很难识破这种共谋。

**目前没有完美解法，只能靠工程手段把风险压低**：
- 尽量把约束下沉到**可机器验证**的地方：类型定义、schema 校验、合约测试、数据库外键约束、OpenAPI spec 生成客户端——这些不会被 AI 的叙述污染
- 定期安排一次 **非 AI 参与的人工 review**：对着代码读一遍，不读文档，看有没有和常识冲突的地方
- 让 harness-audit 类 agent 专门扫**"文档描述了但代码里找不到"**的差集
- 接受一个事实：**文档越完备，越容易让所有人（包括你自己）以为代码一定符合文档**。警惕这种心理舒适感

</details>

<details markdown="1">
<summary>2. 六层资产长期维护的成本是非线性的</summary>

初始化成本是半天到一天，但**维护成本会随项目时间线上升**：

- L1 规则会漂移（技术栈升级、惯例变了），但没人主动更新，AI 按过时规则写代码
- L3 Skill 超过一定数量后，`description` 之间互相抢触发，选用开始随机
- L5 Auto Memory 里会残留过时条目（修好的 bug、换了的依赖），Claude 不会主动清理
- L6 定时任务会出现告警疲劳，报告没人看

harness-audit agent 能部分缓解，但**它本身也需要维护和校准**——谁监督监督者是开放问题。目前我的做法是每 1-2 个月手动 review 一次各层资产，裁剪过期内容；但这套"维护的维护"没有形成稳定流程。

</details>

<details markdown="1">
<summary>3. 认知摩擦高，传播门槛陡</summary>

整套方法论要求读者同时消化：六层资产、两段工作流、两类文档、三类记忆、五步顺序、Hook/Skill/Agent 的各自定位……

**概念量超过大多数人的工作记忆**。这也是为什么上面专门写了"最小起步版本"——先把核心抽出来，剩下的按痛点渐进引入。

即便如此，这套做法的**潜在受众就是比较窄**的：中长期项目 + 愿意前期投入 + 有流程意识的人。大多数用 Claude Code 的人只想"再快一点"，这套不对他们的口味。**方法论不需要适合所有人**——你是不是目标用户，看起因那几条痛点有没有击中你。

</details>

<details markdown="1">
<summary>4. 团队场景没有被充分考虑</summary>

这套做法是基于单人或 2-3 人紧密小团队的实践总结。**超过这个规模，以下几点会出问题**：

- 不同成员对"规则颗粒度 / 架构文档颗粒度"偏好不一致 → L1 / 类别 A 文档会变成战场
- 每个人的 Auto Memory 独立积累 → 同一项目、A 的 Claude 和 B 的 Claude 学到的东西不一样
- 工作区记录是个人的还是共享的？方法论没给明确答案
- 多个人的 review agent 各跑一遍，结论不一致时怎么仲裁？

在中型团队（5+ 人）落地前，这些需要先补一套同步机制。目前这套方法论对团队场景不保证有效。

</details>

---

## 这套做法带来的变化

定量数字我没做过严谨统计，以下是大概感受：

- **单个需求从开始到归档**：前期多花一点时间（写需求分析），但**事后返工**明显少。总账算下来省时间。
- **跨会话 / 跨 AI 交接**：基本不再有"得先花 20 分钟重新讲进度"这种事
- **新人（或隔几个月回来的自己）上手**：读完项目地图 + 最近几条变更记录 + 一两份架构文档，很快能进入状态
- **跨项目迁移**：L1 规则和 L3 自写 Skill 几乎原样 `cp` 过去，省掉新项目头一天的配置时间

---

## 还在探索的方向

诚实说几条我没想清楚的（和上一节"已知的局限"不同——那节是**可能永远解不了的**，这节是**我还在摸索但倾向找到答案的**）：

<details markdown="1">
<summary>点开看我还在摸索的 7 个问题</summary>

1. **TDD 要不要更严**：superpowers 强制 TDD（先写测试再写实现）。我目前的做法是"需求分析里写测试清单 + 编码时同步写测试"，不强制先红后绿。哪种更合适还在看。
2. **工作区记录和变更记录能不能合并**：目前是两份。理论上工作区记录归档后就能变成变更记录的一部分，但自动合并容易错，手动合并太烦。
3. **架构文档什么颗粒度合适**：每个子模块一份？每个主要类一份？没统一标准，目前靠感觉。
4. **本地定时任务 vs 云端 Routines 怎么分工**：需要访问本机未提交文件 / 本机跑着的服务，只能用本地定时任务，但电脑关机会漏；只需要 repo 内容、希望电脑关机也跑就选 Routines。两者怎么结合成一套稳定的自动巡检体系，还在摸索。
5. **协议化 vs 保持随意**：这套做法做成可分发的东西（插件 / CLI / 模板），哪些该硬约束，哪些该留给自定义？一上来就定协议风险大；完全不约束又失去可复用性。
6. **非平凡变更要不要自动生成 walkthrough**：我越来越觉得这很值。让 fresh agent 写一份短的代码走读，说明关键文件、数据流、边界条件和测试证明，几个月后回看会轻很多。
7. **review 要不要彻底分离上下文**：现在我越来越倾向让 review Agent 用 fresh context，只看代码、设计和测试，不看实现会话的过程记录。

</details>

---

## 结语

这套做法是我自己的实战总结，不保证适合所有项目。写出来一方面帮自己梳理清楚，一方面看看社区有没有人做类似的事。

特别想听：
- 你怎么组织架构文档？每个模块一份还是整体一份？
- 你有没有"工作区记录"这种跨会话接力的做法？
- 有哪些社区 Skill / Plugin 是你觉得必装的？

如果这套做法对你有帮助，或者你有类似的打法，欢迎在评论区 / issue 区分享你的实际经验。

---

## 参考资料

**Claude Code 官方文档**
- [How Claude remembers your project](https://code.claude.com/docs/en/memory) — CLAUDE.md 与 Auto Memory
- [Create custom subagents](https://docs.anthropic.com/en/docs/claude-code/agents) — 子 Agent 配置与持久记忆（`memory: user/project/local`）
- [Plugins reference](https://code.claude.com/docs/en/plugins-reference) — Plugin / Skill / Agent / Hook 的权威 schema
- [Schedule recurring tasks in Desktop](https://code.claude.com/docs/en/desktop-scheduled-tasks) — 本地定时任务
- [Automate work with routines](https://docs.anthropic.com/en/docs/claude-code/web-scheduled-tasks) — 云端 Routines
- [Use Claude Code with Chrome](https://docs.anthropic.com/en/docs/claude-code/chrome) — 浏览器自动化

**跨工具标准**
- [AGENTS.md](https://agents.md/) — OpenAI 发起、Linux Foundation 治理的跨工具项目规则文件标准

**相关开源项目**
- [obra/superpowers](https://github.com/obra/superpowers) — 本文主要思想来源之一
- [anthropics/skills](https://github.com/anthropics/skills) — Anthropic 官方维护的 Skill 仓库
- [Pimzino/claude-code-spec-workflow](https://github.com/Pimzino/claude-code-spec-workflow) — 另一套完整的 spec-driven 工作流实现

**外部方法论 / 实践**
- [How OpenAI uses Codex](https://openai.com/business/guides-and-resources/how-openai-uses-codex/) — OpenAI 团队对 Codex 的日常用法和最佳实践
- [Codex use cases](https://developers.openai.com/codex/use-cases) — OpenAI 对 code review、repo understanding、skills 等场景的整理
- [Agentic Engineering Patterns](https://simonwillison.net/guides/agentic-engineering-patterns/) — Simon Willison 的 coding-agent 方法论汇总
- [Repository map](https://aider.chat/docs/repomap.html) — Aider 的 repo map 思路
- [Linting and testing](https://aider.chat/docs/usage/lint-test.html) — Aider 的自动 lint / test 机制
- [Spec Kit](https://github.github.com/spec-kit/) — GitHub 的 spec-driven development 工具链
