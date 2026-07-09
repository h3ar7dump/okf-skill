# OKF Skill —— 用 Markdown 构建可维护的知识图谱

> 一个面向 Claude / Anthropic 智能体的 [Agent Skill](https://docs.claude.com/en/docs/agents-and-tools/agent-skills)，用于构建、浏览、增补与校验 **OKF（Open Knowledge Format，开放知识格式）** 知识包。

[![OKF Spec](https://img.shields.io/badge/OKF%20Spec-v0.1%20Draft-blue)](https://github.com/GoogleCloudPlatform/knowledge-catalog/blob/main/okf/SPEC.md)
[![License](https://img.shields.io/badge/license-MIT-green)](#许可证)

---

## 目录

- [这是什么](#这是什么)
- [为什么用 OKF](#为什么用-okf)
- [仓库结构](#仓库结构)
- [安装 Skill](#安装-skill)
- [如何使用 Skill](#如何使用-skill)
- [四个核心工作流](#四个核心工作流)
- [快速上手示例](#快速上手示例)
- [核心概念速查](#核心概念速查)
- [参考资料](#参考资料)
- [贡献](#贡献)
- [许可证](#许可证)

---

## 这是什么

本仓库提供一个 **Agent Skill**（智能体技能），它教会 Claude 如何把一份知识领域（数据集、API、代码库、文档语料）整理成一个**自描述的、可维护的 Markdown 知识图谱**——也就是一个 **OKF bundle（知识包）**。

一个 OKF bundle 就是一个目录，里面装着一批 Markdown 文件。每个文件是一个 **concept（概念）**，顶部带 YAML frontmatter，文件之间用普通的 Markdown 链接互连，从目录树变成一张图。**没有 SDK，没有运行时，没有锁定**：能 `cat` 就能读，能 `git clone` 就能分发。

这个 skill 把这套规范变成 Claude 可执行的、可复用的工作方法：什么时候建、怎么建、怎么读、怎么维护、怎么体检。

## 为什么用 OKF

知识库大多死于**维护负担**——交叉引用要更新、新旧资料会矛盾、一次改动要牵动十几个页面，这些“记账工作”增长得比价值还快，人就放弃了。

OKF 的思路（源自 [LLM-Wiki 模式](#参考资料)）是：**把维护交还给不会疲倦的智能体**。它不会忘记一个交叉引用，能在一次任务里触及十几个文件，把维护成本压向零。本 skill 正是按这个理念设计的：把 bundle 当作**会生长的知识代码库**，而不是一次性的转储。

OKF 本身遵循三条设计原则：

- **最小化约束** —— 只规定让语料“自描述”所必需的一小撮结构约定（核心是每个概念必须带一个 `type`）。不定义类型分类法、不规定正文小节、不规定存储与查询基础设施。
- **格式而非平台** —— 它是一份开放规范，不是服务、云、数据库或框架。永远不会要求专有账号或 SDK。
- **宽松消费** —— bundle 通过三条结构规则即视为合规；消费方不得因缺字段、未知类型、未知键、断链或缺失索引而拒绝它。这是故意的，因为 bundle 会增长、会被重构、会由智能体部分生成。

## 仓库结构

```
.
├── README.md                # 你正在读的文件
├── LICENSE                  # MIT 许可证
├── .gitignore
└── skills/
    └── okf/
        ├── SKILL.md         # Skill 本体：规范 + 工作流（模型读取它来干活）
        └── GLOSSARY.md      # 术语表：OKF 领域模型与设计原则的权威定义
```

- **`SKILL.md`** —— skill 的入口。顶部 frontmatter 的 `description` 告诉模型**何时**触发该 skill；正文给出 Produce（构建）、Consume（浏览问答）、Enrich（增补）、Lint（体检）四套工作流，以及 frontmatter、保留文件名、链接、合规规则等参考。
- **`GLOSSARY.md`** —— 术语词典。`SKILL.md` 中出现的**粗体术语**都在此精确定义，按“结构 / 导航 / 角色 / 原则 / 渊源”五个轴线组织。当你需要某个规则的来龙去脉时读它。

## 安装 Skill

本 skill 遵循 [Anthropic Agent Skills](https://docs.claude.com/en/docs/agents-and-tools/agent-skills) 规范。把 `skills/okf/` 目录放到 Claude 能发现的位置即可，无需构建、无需依赖。

**方式一：通过 Claude Code 插件市场安装（推荐，最简单）**

如果你的 Claude Code 版本支持插件市场（`/plugin` 命令），直接把本仓库添加为 marketplace 并安装即可，无需手动复制文件：

```shell
/plugin marketplace add h3ar7dump/okf-skill
/plugin install okf@okf-skill
```

安装后，Claude 会自动发现 `okf` skill 并按需触发，无需任何激活命令。后续更新用 `/plugin marketplace update` 与 `/plugin update okf@okf-skill` 拉取最新版本。

> 本仓库根目录的 `.claude-plugin/marketplace.json` 就是插件市场清单：marketplace 名为 `okf-skill`，插件名为 `okf`，所以安装命令是 `okf@okf-skill`。

**方式二：作为项目级 skill（随项目分发）**

在你的项目根目录下：

```bash
# 如果你已 clone 本仓库到本地
cp -r /path/to/this-repo/skills/okf  .claude/skills/okf

# 或用 git submodule 持续跟踪
git submodule add <本仓库地址> .claude/skills/okf-upstream
cp -r .claude/skills/okf-upstream/skills/okf .claude/skills/okf
```

目录结构应为：

```
你的项目/
└── .claude/
    └── skills/
        └── okf/
            ├── SKILL.md
            └── GLOSSARY.md
```

**方式三：作为用户级 skill（所有项目可用）**

```bash
# Linux / macOS
cp -r skills/okf  ~/.claude/skills/okf

# Windows (PowerShell)
Copy-Item -Recurse skills/okf  $HOME\.claude\skills\okf
```

**方式四：直接引用原始文件**

如果你只想读懂规范、自己实现工具，直接阅读 [`skills/okf/SKILL.md`](skills/okf/SKILL.md) 和 [`skills/okf/GLOSSARY.md`](skills/okf/GLOSSARY.md) 即可——它们就是全部内容，没有隐藏的运行时。

> 安装后无需任何命令激活。Claude 会根据你的请求内容，自动判断是否需要调用本 skill（详见下一节的触发关键词）。

## 如何使用 Skill

Skill 一旦被 Claude 发现，就会在你**用自然语言描述任务**时按需自动触发——你不需要记命令。

### 1. 触发它

只要你的话里出现这些线索，Claude 就会加载本 skill：`OKF`、`Open Knowledge Format`、`知识包`、`knowledge bundle`、`知识目录`、`knowledge catalog`、`LLM wiki` 等。

例如直接对 Claude 说：

- “帮我把这个 BigQuery 数据集整理成一个 OKF 知识包。”
- “基于 `/api` 下的代码，生成一个 OKF bundle 作为团队的 LLM wiki。”
- “读一下这个 bundle，回答：orders 表和 customers 表是什么关系？”
- “给这个 bundle 加一个 `metrics/revenue` 概念，并接好交叉链接。”
- “体检一下这个 bundle，列出孤儿概念和断链。”

### 2. Claude 会怎么做

skill 定义了**三个始终分离的层**，Claude 会严格遵守：

- **原始资料（Raw sources）** —— 数据集、文档、代码、schema、录音稿。不可变，是事实依据。Claude 只读不写。
- **bundle（知识包）** —— Claude 作者拥有并维护的 Markdown wiki。每条事实主张都来自原始资料并被**引用**。
- **约定（Conventions）** —— 本 skill 与 OKF spec，即下面的规则。

还有一条凌驾一切的硬规则：**每个概念文件都必须带一个非空的 `type` 字段**。其余都是产出方的自由；消费方容忍其余一切。

### 3. 验证是否生效

你可以让 Claude 复述它将遵循的规则，例如：“按 OKF 规范，一个概念文件的 frontmatter 必须有哪些字段？” 合规的回答应指出 `type` 是唯一必填项，其余为推荐。

## 四个核心工作流

`SKILL.md` 把所有操作归纳为四条工作流，每条都有明确的“完成标准（Done when）”。

| 工作流 | 触发场景 | Claude 做的事 |
|--------|----------|---------------|
| **Produce（构建）** | 从零创建 bundle | 侦察领域 → 设计目录树 → 起草概念 → 连接成图 → 建索引 → 校验 |
| **Consume（浏览问答）** | 从已有 bundle 回答问题 | 先读根 `index.md` → 逐层下钻 → 读概念 → 跟随交叉链接 → 综合作答并按概念 ID 引用 |
| **Enrich（增补）** | 给已有 bundle 增量更新 | 读目标及其邻域 → 更新 frontmatter/正文 → 补链接 → 更新受影响的 `index.md` → 写 `log.md` |
| **Lint（体检）** | 健康检查 | 合规检查（三条规则）+ 图与语义检查（孤儿、断链、矛盾、过时主张、缺口） |

**Produce 的六个步骤**值得单独一提，它是构建任何 bundle 的标准流程：

1. **侦察领域** —— 列出所有值得记录的来源，为每个挑一个 `type`（如 `BigQuery Table`、`API Endpoint`、`Metric`、`Playbook`）。类型无需集中注册，按领域命名即可。
2. **设计目录树** —— 给每个概念一个文件路径，相关概念归入复数命名的子目录（`tables/`、`metrics/`、`playbooks/`）。路径就是身份：去掉 `.md` 即**概念 ID**（`tables/orders.md` → `tables/orders`）。不得使用保留文件名 `index.md`、`log.md`。
3. **起草概念** —— 先 YAML frontmatter，再 Markdown 正文。正文偏好结构化（标题、列表、表格、代码块）而非散文。
4. **连接成图** —— 在相关概念间加链接，优先用**绝对包相对路径**（以 `/` 开头），因为它在概念跨目录移动后仍然有效。
5. **建立索引** —— 每个目录放一个 `index.md`，分节列出该目录内容，每条带上对应概念的 `description`。索引文件不带 frontmatter（bundle 根目录的 `index.md` 可声明 `okf_version: "0.1"`）。
6. **校验** —— 跑一遍 v0.1 合规检查，修复或记录为有意偏离。

## 快速上手示例

假设你有一个电商数据集，想把它做成知识包。对 Claude 说：

> “读取 `schema/` 下的表结构 SQL 和 `docs/api.md`，按 OKF 规范生成一个知识包。包含 `tables/`（每张表一个概念，带 `# Schema` 小节）、`metrics/`（关键指标，带定义与计算口径）、`endpoints/`（API 端点）。根目录放 `index.md`，并在根 `index.md` 声明 `okf_version: "0.1"`。完成后做一次 lint。”

Claude 会产出类似这样的结构：

```
my-bundle/
├── index.md                 # 全局目录，声明 okf_version: "0.1"
├── log.md                   # 变更历史，按 YYYY-MM-DD 倒序
├── tables/
│   ├── index.md
│   ├── orders.md            # frontmatter: type: BigQuery Table
│   └── customers.md         # frontmatter: type: BigQuery Table
├── metrics/
│   ├── index.md
│   └── revenue.md           # frontmatter: type: Metric
└── endpoints/
    ├── index.md
    └── create-order.md      # frontmatter: type: API Endpoint
```

一个概念文件的样貌：

```markdown
---
type: BigQuery Table
title: Orders
description: 每笔已完成订单一行，覆盖所有渠道
resource: bigquery://project.dataset.orders
tags: [电商, 订单]
timestamp: 2026-07-09T00:00:00Z
---

# Orders

每行代表一笔跨渠道的已完成客户订单。与 [/tables/customers.md](/tables/customers.md)
通过 `customer_id` 关联。

# Schema
| 列名 | 类型 | 说明 |
|------|------|------|
| order_id | STRING | 订单唯一标识 |
| customer_id | STRING | 外键 → [/tables/customers.md](/tables/customers.md) |
| placed_at | TIMESTAMP | 下单时间 |

# Citations
[1] schema/orders.sql
```

> 注意链接用绝对包相对路径（`/tables/customers.md`），即使文件日后移到别的子目录也不会断。

## 核心概念速查

完整定义见 [`skills/okf/GLOSSARY.md`](skills/okf/GLOSSARY.md)。这里是最常用的几个：

- **Bundle（知识包）** —— 自包含的、层级化的概念目录，是分发单位。一个知识域一个 bundle。
- **Concept（概念）** —— bundle 内一个知识单元，即一个 Markdown 文件。文件路径即其身份。
- **Concept ID（概念 ID）** —— 文件路径去掉 `.md`。引用概念时用 ID。
- **Frontmatter** —— 概念文件顶部 `---` 包裹的 YAML 块。`type` 必填，`title`/`description`/`resource`/`tags`/`timestamp` 推荐。
- **Index File（索引文件）** —— `index.md`，分节列出目录内容，实现渐进式披露。
- **Log File（日志文件）** —— `log.md`，按 ISO 日期倒序记录变更。
- **Link（链接）** —— 普通 Markdown 链接，断言一条**无类型关系**；关系的种类由周围散文传达。消费方必须容忍断链。
- **Citation（引用）** —— 指向外部来源的链接，在 `# Citations` 下编号列出，用来让 bundle 有据可依、而非凭空捏造。

**合规（v0.1）三条规则**：① 每个非保留 `.md` 文件都有可解析的 frontmatter；② 每个 frontmatter 都有非空 `type`；③ 每个存在的 `index.md`/`log.md` 都遵循规定结构。

## 参考资料

- **OKF 规范（v0.1 Draft）** —— 本 skill 的规范依据。仓库：[GoogleCloudPlatform/knowledge-catalog](https://github.com/GoogleCloudPlatform/knowledge-catalog)，规范文件：[`okf/SPEC.md`](https://github.com/GoogleCloudPlatform/knowledge-catalog/blob/main/okf/SPEC.md)。
- **LLM-Wiki 模式** —— Andrej Karpathy 提出的模式：让 LLM 增量地构建并维护一份位于你和原始资料之间的、持久且复利的 Markdown wiki。OKF 即此模式的规范化。原文：[Karpathy 的 LLM Wiki gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)。
- **Anthropic Agent Skills 文档** —— skill 文件格式（frontmatter 的 `name`/`description`、目录约定）的官方说明：<https://docs.claude.com/en/docs/agents-and-tools/agent-skills>。
- **本仓库内部文档** —— [`SKILL.md`](skills/okf/SKILL.md)（规范与工作流）、[`GLOSSARY.md`](skills/okf/GLOSSARY.md)（术语与设计原则）。

## 贡献

欢迎提 Issue 和 PR。如果你为某个新领域产出了 bundle，或在别的智能体/工具里实现了 OKF 消费端，欢迎在 Issue 里分享。

改动 `SKILL.md` / `GLOSSARY.md` 时，请保持术语一致（`GLOSSARY.md` 是术语权威来源，`SKILL.md` 中的**粗体术语**都应能在其中找到定义）。

## 许可证

本 skill 仓库以 MIT 许可证发布。OKF 规范本身的许可证以上游 [knowledge-catalog](https://github.com/GoogleCloudPlatform/knowledge-catalog) 仓库为准。
