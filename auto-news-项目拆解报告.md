# finaldie/auto-news 项目拆解报告

> 说明：当前仓库没有 `network.py`、`config.py`、`pipeline.py`、`summarize.py` 这些文件；对应职责主要分布在 `src/af_pull.py`、`src/af_save.py`、`src/ops_*.py`、`src/llm_agent.py`、`src/llm_prompts.py`、`dags/*.py` 和 `.env.template` 中。

## 1. 项目一句话定位

`auto-news` 是一个面向个人的信息聚合与 LLM 增强处理系统：它把 RSS、Twitter、Reddit、网页文章、YouTube 等信息源自动拉取进来，通过去重、摘要、评分、分类、排序等流程，最终沉淀到 Notion 这样的个人知识/阅读工作台中。

它解决的核心痛点是：信息源太分散、噪音太多、人工筛选成本高，用户需要一个自动帮自己“收集、过滤、总结、排序、复盘”的 AI 信息助理。

## 2. 目标用户与使用场景

目标用户主要是高信息密度工作者，例如：

- AI/科技从业者：需要持续追踪论文、产品、行业动态、技术趋势。
- 独立开发者/创业者：需要从社交媒体、社区、新闻源中快速发现机会。
- 内容创作者/研究者：需要定期收集素材、生成洞察、做周报复盘。
- 重度 Notion 用户：希望把阅读、收藏、总结、行动项统一放进一个知识库。

典型使用场景：

- 每小时自动抓取新内容，推送到 Notion 阅读库。
- 从大量 Tweet、Reddit 帖子、RSS 文章里过滤掉低相关内容。
- 对长文章、YouTube 视频、Reddit 长帖生成摘要。
- 根据用户过往评分，用向量相似度判断新内容是否符合兴趣。
- 每周生成 Top-K 复盘内容。
- 从 Takeaways、Journal 中生成 TODO/action items。

## 3. 核心用户流程

用户先在 Notion 中维护自己的信息源，例如 RSS 列表、Twitter 关注列表、Reddit subreddit 列表、文章/YouTube 收件箱等。同时配置 LLM provider、embedding provider、Notion token、数据库连接等环境变量。

Airflow 定时触发任务。系统先执行拉取阶段，把不同来源的新内容抓取下来，并保存为中间 JSON。然后执行处理阶段，对内容做去重、相似度评分、过滤、摘要、分类、排序，并缓存 LLM 结果，避免重复调用。

处理后的内容被写入 Notion 的 ToRead 数据库，带有标题、来源、链接、摘要、主题、分类、评分、takeaways 等结构化字段。用户最终在 Notion 中像使用一个智能 RSS Reader/知识库一样浏览结果。

## 4. 模块与架构拆解

### 输入接入层：如何获取信息

核心文件：

- `src/af_pull.py`
- `src/ops_rss.py`
- `src/ops_article.py`
- `src/ops_youtube.py`
- `src/ops_twitter.py`
- `src/ops_reddit.py`
- `src/notion.py`

输入来源包括：

- RSS：通过 `feedparser` 解析 RSS feed。
- Twitter：通过 Twitter API 拉取指定列表账号的推文。
- Reddit：通过 Reddit API 拉取 subreddit posts。
- Article：从 Notion Inbox 中读取用户收藏的文章 URL，再加载网页内容。
- YouTube：从 Notion Inbox 中读取视频链接，提取 transcript；没有 transcript 时会尝试音频转写相关流程。
- Journal/Takeaways：从 Notion 中读取用户笔记，用于 TODO 和复盘生成。

这一层的职责是把不同形态的信息源统一转成内部 page/post/tweet 数据结构，并保存为 JSON 中间结果。

它解决的核心问题是：不同平台 API、字段、内容结构完全不同，需要先抽象成统一的“待处理内容”。

### 数据清洗层：去噪、去重、标准化结构

核心文件：

- `src/ops_base.py`
- `src/db_cli.py`
- `src/data_model.py`
- `src/ops_milvus.py`
- 各 `Operator*.dedup/filter/score` 方法

主要机制：

- Redis/MySQL 记录已处理 item id，避免重复写入 Notion。
- 对 Article/YouTube/RSS 使用 page id 或 hash id 去重。
- 对 Twitter/Reddit 使用 tweet id、post hash id 去重。
- RSS 通过 `list_name + title + published_key` 生成 md5，降低 RSS 日期抖动导致的重复。
- Twitter/Reddit/RSS 会使用 Milvus 向量检索，基于用户历史评分内容计算相关性分数。
- 根据不同列表配置不同过滤阈值，例如 `TWITTER_FILTER_MIN_SCORES`、`REDDIT_FILTER_MIN_SCORES`。

这一层的职责是减少重复内容和低价值内容，让 LLM 只处理更可能有用的候选项。

它解决的核心问题是：LLM 调用成本高，不能把所有原始信息都直接送进模型。

### LLM 调用层：摘要、标签、评分、行动项

核心文件：

- `src/llm_agent.py`
- `src/llm_prompts.py`
- `src/ops_article.py`
- `src/ops_youtube.py`
- `src/ops_rss.py`
- `src/ops_reddit.py`
- `src/ops_twitter.py`
- `src/ops_todo.py`
- `src/ops_journal.py`

LLM 支持：

- OpenAI
- Google Gemini
- Ollama

主要调用模式：

- 摘要：`LLMAgentSummary` 使用 LangChain 的 `load_summarize_chain`，采用 `map_reduce`，先切块再总结。
- 分类与评分：`LLMAgentCategoryAndRanking` 要求模型输出 JSON，包括 topics、category、overall_score、feedback。
- 翻译：通过 `TRANSLATION_LANG` 控制是否生成翻译版本。
- Journal 整理：把零散日记整理成结构化笔记、洞察、takeaways、TODO。
- TODO 生成：从 takeaways 和 journal 中抽取 action items。

这一层的职责是把原始内容变成“可阅读、可筛选、可行动”的结构化信息。

它解决的核心问题是：信息不仅要被收集，还要被压缩、解释、打标签，并转换成下一步行动。

### 输出层：结果结构、文件/数据库/看板形式

核心文件：

- `src/af_save.py`
- `src/notion.py`
- `src/ops_notion.py`
- `src/ops_collection.py`
- `src/af_publish.py`

输出形式包括：

- 中间 JSON 文件：用于 Airflow 拉取阶段和保存阶段之间传递数据。
- Notion ToRead 数据库：最终主要阅读界面。
- Notion Inbox 数据库：用户输入配置和手动收藏入口。
- Redis/MySQL：保存去重状态、时间游标、LLM 缓存、用户评分元数据。
- Milvus：保存内容 embedding，用于相关性检索和兴趣匹配。
- Weekly/Monthly Collection：从高评分内容中做周期复盘。

这一层的职责是把 AI 处理结果落到用户已有的工作台里，而不是只停留在命令行或临时输出。

它解决的核心问题是：信息处理必须形成长期可回看的知识资产。

## 5. 核心数据流与调用链

完整链路可以理解为：

1. 用户在 Notion Inbox 中配置 RSS/Twitter/Reddit/Article/YouTube 来源。
2. Airflow 的 `news_pulling` DAG 每小时触发。
3. `af_pull.py` 根据 `CONTENT_SOURCES` 依次调用对应 Operator。
4. 每个 Operator 从 Notion/API/RSS/网页/视频平台拉取原始数据。
5. 拉取结果保存成 JSON，例如 `twitter.json`、`rss.json`、`article.json`。
6. `af_save.py` 读取 JSON 中间结果。
7. 系统先用 Redis/MySQL 状态判断是否重复。
8. 对 Twitter/Reddit/RSS 等高噪声来源，先用 embedding + Milvus 做相关性评分。
9. 按分数阈值过滤低价值内容。
10. 对文章、视频、RSS、长帖等内容调用 LLM 生成摘要。
11. 对摘要或短文本调用 LLM 做主题、分类、质量评分。
12. 解析 LLM JSON 输出，抽取 topics、categories、overall score。
13. 将最终内容写入 Notion ToRead 数据库。
14. 用户在 Notion 中阅读、评分、沉淀 takeaways。
15. 后续同步任务把用户评分内容写入 Milvus，反过来优化下一轮筛选。

## 6. 关键设计亮点

### 1. Operator 模式清晰

每个信息源都有独立 `Operator`，并实现类似的生命周期：

```text
pull -> dedup -> score/filter -> summarize -> rank -> push
```

这让 RSS、Twitter、Reddit、Article、YouTube 虽然来源不同，但处理范式一致，扩展新数据源比较自然。

### 2. 把 Notion 同时作为输入端和输出端

Notion 不只是展示页面，也承担了配置中心、Inbox、ToRead、用户评分入口的角色。这是很实用的产品设计：用户不需要额外后台，就能通过 Notion 管理系统。

### 3. LLM 调用前先做过滤

项目没有直接把所有内容丢给 LLM，而是先去重、再用 embedding 和历史用户评分做相关性筛选。这对真实产品很重要，因为 LLM 成本、速度、失败率都不可忽视。

### 4. 用户反馈闭环

用户在 Notion 中给内容评分后，系统会同步这些评分，并通过 Milvus 相似度影响未来内容筛选。这让系统从“静态规则过滤器”变成“可学习的个人兴趣模型”。

### 5. 多 LLM 后端可替换

通过 `LLM_PROVIDER` 支持 OpenAI、Google、Ollama。摘要、分类、翻译等能力封装在 `llm_agent.py`，没有散落在所有业务代码里。

### 6. 有生产化调度意识

项目使用 Airflow DAG、Docker Compose、Helm、ArgoCD，说明它不是一次性脚本，而是按长期运行的自动化系统设计。

## 7. 限制与局限

### 1. 对中文内容不是一等公民

默认 prompt、README、评分逻辑、摘要格式主要面向英文内容。虽然支持 `TRANSLATION_LANG`，但这更像“翻译补丁”，不是完整中文语义优化。

对于中文内容，可能存在：

- 中文网页正文抽取质量不稳定。
- 中文标题、话题、分类的 prompt 不够本地化。
- 中文社媒语境、招聘术语、行业黑话识别不足。
- Notion 多选标签长度限制会影响中文标签设计。

### 2. 国内信息源适配不足

当前更偏国际信息源：

- Twitter
- Reddit
- YouTube
- RSS
- Arxiv
- Web Articles

对国内求职/AI 产品研究常用来源支持不足，例如：

- Boss 直聘
- 拉勾
- 猎聘
- 脉脉
- 小红书
- 微信公众号
- 知乎
- 即刻
- 国内公司招聘官网

### 3. 抓取与解析鲁棒性有限

网页正文依赖 LangChain `WebBaseLoader` 等通用工具，对反爬、动态渲染、登录页、中文站点结构适配有限。

### 4. LLM 输出 JSON 仍有解析风险

虽然 prompt 要求输出 JSON，并用 `fix_and_parse_json` 修复，但这类方案仍可能在复杂内容下失败。更稳的做法是使用结构化输出、schema validation 或 function calling。

### 5. 配置复杂，普通用户部署门槛高

系统依赖 Airflow、Redis、MySQL、Milvus、Notion API、LLM API、多个第三方平台 API。对个人用户来说，自托管门槛偏高。

### 6. 产品界面依赖 Notion

Notion 适合作为快速工作台，但如果要做面向求职者或团队的正式产品，需要更明确的权限、看板、检索、对比、任务管理和数据可视化能力。

## 8. 对 AI Workbench / 求职辅助产品的启发

### 1. 可以借鉴“多源输入 -> 标准化内容对象”的架构

你的 AI Workbench 可以把 JD、公司页面、招聘平台岗位、公众号文章、面试经验、简历版本都抽象成统一对象：

```text
source
title
url
company
role
content
created_time
tags
summary
score
matched_skills
action_items
```

这样后续无论是 JD 分析、简历匹配、求职复盘，底层流程都能复用。

### 2. 用“用户评分/行为”反向训练筛选逻辑

`auto-news` 的亮点是用户评分进入 Milvus，再影响下一轮推荐。你的产品也可以这样设计：

- 用户收藏某类 JD。
- 用户标记“值得投”“不匹配”“薪资不合适”“技术栈有价值”。
- 系统把这些 JD embedding 化。
- 新 JD 进入后，先和历史高价值 JD 做相似度匹配。
- 再决定是否进入 LLM 深度分析。

这能减少无效岗位分析成本。

### 3. LLM 不要直接处理全部信息，先做轻量筛选

对求职场景尤其重要。可以分层：

- 第一层：规则过滤，如城市、薪资、经验、学历、公司规模。
- 第二层：embedding 匹配，如岗位职责与简历经历相似度。
- 第三层：LLM 深度分析，如 JD 拆解、简历改写、面试问题预测。

这比“所有 JD 都调用大模型”更可控。

### 4. 把输出设计成“可行动工作台”

`auto-news` 不只是生成摘要，还生成 TODO、takeaways、weekly recap。你的 AI 求职产品也应该避免只输出分析报告，可以输出：

- 今日优先投递岗位
- 每个岗位的简历修改建议
- 缺口技能清单
- 面试准备问题
- 投递状态
- 跟进提醒
- 一周求职复盘

也就是从“信息解释器”升级为“求职执行系统”。

### 5. Notion 思路可以迁移为 MVP

如果你还在早期阶段，不一定要一开始做完整 Web App。可以先用 Notion/飞书表格/Airtable 做前端工作台，后端只负责：

- 抓取 JD
- 分析 JD
- 写回结构化字段
- 生成匹配建议
- 记录用户反馈

这会比直接做完整 SaaS 更快验证产品逻辑。

## 9. 可以写进简历/作品集的表达模板

1. 深度拆解开源 AI 信息聚合项目 `auto-news`，分析其多源采集、去重过滤、LLM 摘要分类、Notion 输出和用户反馈闭环架构，沉淀为可复用的 AI Workbench 产品设计方法。

2. 研究基于 Airflow 的自动化内容处理流水线，梳理 `pull -> dedup -> score -> summarize -> rank -> push` 的模块化流程，并总结其在求职 JD 分析系统中的迁移方案。

3. 分析项目如何结合 Redis/MySQL/Milvus 实现状态管理、LLM 缓存、内容去重和个性化相关性评分，为后续构建中文岗位匹配与简历优化系统提供架构参考。

4. 对比 RSS、Twitter、Reddit、YouTube、网页文章等不同输入源的接入方式，抽象出统一内容对象模型，用于设计多源中文招聘信息聚合与分析工作台。

5. 基于 `auto-news` 的 Notion 工作台模式，总结“信息输入、AI 处理、结构化输出、用户反馈再训练”的产品闭环，并迁移到 AI 求职辅助产品的功能规划中。
