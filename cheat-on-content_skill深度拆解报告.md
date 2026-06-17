# XBuilderLAB/cheat-on-content Skill 深度拆解报告

> 分析对象：[XBuilderLAB/cheat-on-content](https://github.com/XBuilderLAB/cheat-on-content)  
> 分析目的：学习它的 workflow 设计、skill 拆分、状态管理和反馈闭环。  
> 注意：本报告只做静态阅读分析，不安装、不运行、不修改项目代码。

---

## 1. 这个 skill 一句话是干嘛的

`cheat-on-content` 是一个把内容创作从“凭感觉发布”改造成“可打分、可预测、可复盘、可升级”的 Agent 工作流系统。

它不是单纯帮你写内容，而是让每一篇内容都变成一次实验：

```text
写稿
  -> 打分
  -> 盲预测
  -> 发布
  -> 回收真实数据
  -> 复盘预测偏差
  -> 升级评分规则
```

真正值得学习的是：它把一个模糊的创作过程，拆成了有状态、有文件、有校验、有反馈的系统。

---

## 2. 核心闭环：score -> predict -> publish -> retro -> bump

### 2.1 总流程

```text
score
  |
  v
predict
  |
  v
publish
  |
  v
retro
  |
  v
bump
  |
  v
新的 rubric
  |
  v
下一篇内容继续循环
```

### 2.2 score：先打分，但不承诺

`cheat-score` 的定位是轻量探索。

用户把一篇草稿交给系统，系统读取：

```text
草稿文件
rubric_notes.md
.cheat-state.json
```

然后输出：

```text
每个维度的分数
每个维度的理由
综合分 composite
下一步建议
```

关键点是：`score` 不写文件、不预测、不改变状态。  
它只是帮用户判断“这篇稿子值不值得进入正式预测流程”。

这点设计很好：把“试试看”与“正式承诺”分开，降低使用压力。

---

### 2.3 predict：正式盲预测，写入不可修改日志

`cheat-predict` 是整个系统的核心。

它做的事不是简单输出一句“我觉得会火”，而是写一份完整预测日志：

```text
输入快照
维度打分
综合分
预测 bucket
概率分布
推理因素
锚点对比
反事实场景
关键校准假设
空的复盘段
```

预测写入 `predictions/*.md` 后，`## 预测` 段原则上不可改。后续只能往 `## 复盘` 段追加。

它的核心设计是：预测必须在看到真实数据之前完成，否则这次预测不再算校准样本。

---

### 2.4 publish：只登记发布元数据

`cheat-publish` 是一个轻量动作，只负责把发布信息补到预测文件和状态文件里。

它记录：

```text
发布时间
平台
URL
平台 ID
对应 video folder
```

同时更新 `.cheat-state.json`：

```text
清除 in_progress_session
写入 last_published_at
加入 pending_retros
从 shoots 队列移除
```

注意它不抓数据。  
这很重要，因为“发布”和“复盘”是两个不同动作。发布当天不应该立刻用真实数据污染预测。

---

### 2.5 retro：T+N 天后回收数据并复盘

`cheat-retro` 是反馈环节。

它在发布后 N 天读取真实表现：

```text
播放 / 阅读
点赞
评论
收藏
转发
top 评论
```

然后对比预测：

```text
哪些判断被验证
哪些判断被推翻
哪些维度高估
哪些维度低估
评论里出现了什么真实信号
```

它会把观察写入 `rubric-memo.md`，而不是直接写进 `rubric_notes.md`。

这个拆分很关键：

- `rubric_notes.md`：给未来盲打分用，只放通用规则。
- `rubric-memo.md`：放真实样本、实绩、评论、证据。

这样可以避免未来盲预测时读到过去真实数据。

---

### 2.6 bump：升级评分规则

`cheat-bump` 是最高风险动作，用来升级 rubric 或重新校准 bucket。

完整 rubric bump 会做：

```text
提出新公式
  -> 校准池全量重打分
  -> 检查新排序是否更贴近真实表现
  -> 跨模型审核
  -> 用户确认
  -> 更新 rubric_notes.md
  -> 写入 rubric-memo.md
  -> 清理旧观察
```

它不是“我感觉这个维度该加权，所以改一下”。  
它要求新规则必须能解释历史数据，否则拒绝升级。

这就是从“提示词”进化到“可验证系统”的关键。

---

## 3. 它为什么强调“盲预测”

盲预测是这个项目的核心纪律。

它强调盲预测，是因为只有在看到真实结果之前写下判断，复盘才有价值。

如果先看到数据再预测，会发生两个问题：

### 3.1 后视镜偏差

看到播放量、评论、点赞之后，模型和人都会不自觉地倒推理由。

比如：

```text
数据很好 -> 觉得标题强、情绪强、选题好
数据很差 -> 觉得结构弱、受众窄、钩子差
```

这种分析看起来很有道理，但它不是预测能力，只是事后解释。

### 3.2 无法校准

这个系统想训练的是：

```text
发布前，我的判断准不准？
```

不是：

```text
结果出来后，我能不能解释它？
```

如果预测可以事后改，那所有复盘都会失去基线。

所以它设计了三层保护：

```text
协议层：blind-prediction-protocol.md
文件层：predictions/*.md 的预测段不可改
hook 层：prediction-immutability.sh 阻止修改预测段
```

这点对你很有启发：  
只要你的产品涉及“判断 -> 结果 -> 复盘”，就要先保护判断记录，不能让它被结果污染。

---

## 4. 它的目录结构怎么组织

项目目录大致分成 9 类。

```text
cheat-on-content/
├── SKILL.md
├── README.md
├── skills/
├── shared-references/
├── templates/
├── hooks/
├── adapters/
├── starter-rubrics/
├── migrations/
├── tools/
├── examples/
└── docs/
```

### 4.1 SKILL.md：总入口和路由器

主 `SKILL.md` 不是具体执行逻辑，而是总协议和路由表。

它负责说明：

```text
这个系统的三条原则
用户说什么话应该路由到哪个子 skill
项目期望的文件结构
哪些请求必须拒绝
```

这是一种很成熟的 skill 设计方式：  
主 skill 不包办所有流程，而是只做“入口、约束、分发”。

---

### 4.2 skills/：子 skill 集合

每个动作都有独立 skill：

```text
cheat-init
cheat-score
cheat-predict
cheat-publish
cheat-retro
cheat-bump
cheat-status
...
```

这种拆法让每个 skill 只负责一个清晰动作，避免一个巨型提示词管所有事情。

---

### 4.3 shared-references/：跨 skill 协议

这里放的是公共协议，比如：

```text
blind-prediction-protocol.md
state-management.md
prediction-anatomy.md
observation-lifecycle.md
bump-validation-protocol.md
cadence-protocol.md
candidate-schema.md
```

这些文件相当于“系统规范”。  
不同子 skill 都引用它们，保证行为一致。

---

### 4.4 templates/：写入用户项目的文件骨架

这里放初始化时会复制到用户项目里的模板：

```text
prediction.template.md
retro.template.md
workflow.template.md
status.template.md
rubric_notes.template.md
rubric-memo.template.md
candidates.template.md
audience.template.md
benchmark.template.md
content.db.schema.sql
```

模板的作用是让用户项目天然形成标准结构。

---

### 4.5 hooks/：工程约束层

hooks 不是内容逻辑，而是强制执行规则。

主要包括：

```text
prediction-immutability.sh
session-start.sh
log-event.sh
```

最重要的是 `prediction-immutability.sh`：  
它拦截对 `predictions/` 下预测段的修改，保证预测不可事后篡改。

---

### 4.6 adapters/：外部数据源适配层

adapters 负责把外部平台数据转成系统能复盘的格式。

目前能看到几类：

```text
perf-data/
trend-sources/
script-extraction/
```

含义分别是：

- `perf-data`：抓发布后的表现数据。
- `trend-sources`：抓热点和候选选题。
- `script-extraction`：从视频/音频提取稿件。

这说明它把“工作流核心”和“平台接入”分开了。

---

### 4.7 starter-rubrics/：初始评分规则

这里放不同内容形态的起步 rubric。

当前重点是观点视频：

```text
opinion-video.md
opinion-video-zero.md
```

它们不是最终真理，只是冷启动规则。  
真正的规则要靠后续 retro 和 bump 慢慢校准。

---

### 4.8 migrations/：状态版本迁移

这个目录说明它把 skill 当成长期维护的软件系统，而不是一次性提示词。

当 `.cheat-state.json` 的 schema 变化时，通过 migration 文件说明：

```text
改了什么
为什么改
怎么迁移
手动兜底方案
```

这是成熟项目才会重视的部分。

---

### 4.9 tools/：辅助脚本

tools 里有一些独立脚本，比如：

```text
score-curve.py
diff_pct.py
diff_pct_test.sh
```

它们不是主流程，但服务于统计、差异计算、质量验证。

---

## 5. 每个子 skill 大概负责什么

| 子 skill | 职责 | 解决的问题 |
|---|---|---|
| `cheat-init` | 初始化项目、创建脚手架、写状态文件、配置 hooks | 让用户从零进入标准工作流 |
| `cheat-learn-from` | 导入对标账号，拆内容 pattern 和初始信号 | 冷启动没有自己历史数据的问题 |
| `cheat-seed` | 和用户一起找选题、生成初稿 | 用户不知道写什么的问题 |
| `cheat-score` | 对单篇草稿打分，只输出不写文件 | 正式预测前的轻量判断 |
| `cheat-score-blind` | 内部盲打分子 agent，只读 script 和 rubric | 防止主对话被历史数据污染 |
| `cheat-predict` | 写不可修改的预测日志 | 建立发布前判断基线 |
| `cheat-shoot` | 登记已拍摄内容，建立 video folder，管理 buffer | 区分“拍了但没发”和“已发布” |
| `cheat-publish` | 登记发布 URL、平台、时间，加入待复盘队列 | 把发布动作纳入状态管理 |
| `cheat-retro` | 回收真实数据，验证/推翻预测，写观察 | 把结果转成可学习信号 |
| `cheat-bump` | 升级 rubric 或重校 bucket | 让系统根据历史表现变聪明 |
| `cheat-status` | 输出当前状态看板 | 告诉用户现在该做什么 |
| `cheat-recommend` | 从候选池推荐下一篇选题 | 把选题变成可排序队列 |
| `cheat-trends` | 抓热点、去重、粗打分、写入候选池 | 解决素材来源问题 |
| `cheat-persona` | 从评论和复盘派生受众画像 | 让系统知道“谁在看” |
| `cheat-migrate` | 升级旧状态文件 schema | 保证长期演进不破坏旧项目 |

它的设计重点不是“有很多命令”，而是每个命令都在闭环中有明确位置。

---

## 6. 状态文件、模板、hooks、adapters 分别有什么作用

### 6.1 状态文件：`.cheat-state.json`

这是整个系统的单一状态源。

它记录：

```text
schema_version
rubric_version
content_form
发布频率
校准样本数
待复盘列表
已拍未发队列
当前预测会话
最近一次发布
最近一次复盘
最近一次 bump
是否安装 hooks
是否跳过 blind sub-agent
```

它的价值是：不同子 skill 不需要互相猜状态，只读写同一个状态文件。

比如：

- `cheat-predict` 写入 `in_progress_session`
- `cheat-publish` 清除 `in_progress_session`，加入 `pending_retros`
- `cheat-retro` 移除 `pending_retros`，增加 `calibration_samples`
- `cheat-bump` 更新 `rubric_version`
- `cheat-status` 汇总这些字段给用户看

这是非常值得你学习的设计：  
AI workflow 只靠对话记忆是不可靠的，必须把关键状态落盘。

---

### 6.2 模板：把流程结构固定下来

模板不是装饰，而是流程约束。

例如 `prediction.template.md` 强制一份预测日志包含：

```text
输入快照
预测
推理因素
锚点对比
反事实场景
关键校准假设
复盘占位
```

这样每次预测都有同样结构，后续 retro 才能对比。

如果没有模板，用户每次写法都不一样，系统就无法稳定复盘。

---

### 6.3 hooks：把“原则”变成工程强制

这个项目最强的点之一是 hook。

它没有只写一句“预测不要改”，而是用 `prediction-immutability.sh` 在工具层拦截修改：

```text
允许：
新建预测文件
修改预测文件顶部 metadata
追加复盘段

禁止：
修改已经写下的预测段
覆盖已有 prediction 文件
```

这说明它知道：靠人的自觉不够，靠模型自律也不够。关键原则要工程化。

---

### 6.4 adapters：平台差异隔离层

adapters 的作用是把平台接入和核心流程解耦。

比如：

```text
抖音数据
B 站数据
小红书数据
微博热点
知乎热点
AI 热点
Whisper 转录
```

这些平台会变，但核心闭环不变：

```text
拿到外部数据
  -> 转成统一 report
  -> retro 读取
  -> 写观察
```

这对你的 AI 电商运营工作台很重要：  
不要一开始把淘宝、抖音、小红书、拼多多逻辑写死在主流程里。应该用 adapter 层隔离。

---

## 7. 最值得学习的 5 个设计点

### 7.1 把模糊工作变成状态机

内容创作原本很模糊：

```text
想到什么写什么
发出去看看
火了就说有感觉
没火就说运气不好
```

它把这个过程变成：

```text
score
predict
publish
retro
bump
```

每一步都有输入、输出、文件、副作用和拒绝条件。

你做 AI Workbench 时也应该这样设计：  
不要只做“AI 帮我分析一下”，而要做“输入什么、生成什么、保存什么、复盘什么”。

---

### 7.2 状态文件是 workflow 的大脑

`.cheat-state.json` 让所有子 skill 共享上下文。

它避免了两个问题：

- 每次对话都要重新解释项目状态。
- 不同模块对“当前进度”理解不一致。

这对你的项目尤其关键。  
求职投递、电商运营、AI Workbench 都需要状态文件。

---

### 7.3 盲预测 + 不可变日志

它没有让 AI 一直“给建议”，而是要求 AI 在结果出现前下注。

这让复盘有了基线。

迁移到你的方向，可以变成：

```text
求职：
投递前预测这个岗位匹配度和回复概率
投递后记录真实结果
复盘偏差

电商：
发布前预测笔记/商品文案表现
发布后记录曝光、咨询、转化
复盘偏差

AI Workbench：
执行任务前预测产出质量和风险
任务后记录真实结果
复盘 workflow 是否要升级
```

---

### 7.4 把“规则”和“证据”分离

它把：

```text
rubric_notes.md
rubric-memo.md
```

拆开。

前者放通用规则，后者放真实证据。

这个设计很成熟，因为未来盲打分时不能读到真实表现数据。

迁移到你的产品，也应该分：

```text
当前规则文件
历史复盘证据文件
```

不要把历史结果、真实反馈、用户评论都塞进未来判断规则里，否则 AI 很容易被污染。

---

### 7.5 bump 不是随便改，而是验证后升级

很多 AI 项目所谓“优化”只是改提示词。

这个项目的 `bump` 要求：

```text
提出新公式
历史样本重打分
排序一致性检查
跨模型审核
用户确认
落地
清理旧观察
```

这给你一个重要启发：  
AI workflow 的迭代不能只靠感觉，要有“升级门槛”。

---

## 8. 它不适合我的地方

### 8.1 它的语境是内容创作者，不是应届生求职

它默认用户持续发布内容，有播放量、评论、粉丝、爆款这些指标。

你的核心目标是：

```text
学习 AI 产品项目
做求职作品集
拆解开源项目
理解国内电商 AI 工具
```

所以你不能照搬它的内容增长语言。

---

### 8.2 它的复杂度偏高

它有：

```text
14+ 子 skill
hooks
adapters
migrations
state schema
blind sub-agent
cross-model audit
```

对你现在来说，不能一上来全做。

你应该先学它的骨架：

```text
结构化输入
预测/判断记录
真实结果回收
复盘
规则升级
```

不要一开始学它的全部工程复杂度。

---

### 8.3 它的指标不直接适配求职和电商

内容创作看：

```text
播放
点赞
评论
转发
收藏
```

求职应该看：

```text
岗位匹配度
是否投递
是否已读
是否约面
面试反馈
是否 offer
```

电商应该看：

```text
曝光
点击
收藏
咨询
加购
成交
退款
客服问题
```

所以不能照搬 rubric，只能照搬 workflow。

---

### 8.4 它依赖持续样本积累

这个系统越用越准。  
但如果用户只有很少样本，早期预测本来就不可靠。

对你来说，求职投递和电商运营也一样：  
没有足够投递记录或运营数据时，AI 只能做辅助判断，不能装成精准预测。

---

### 8.5 它更适合严肃长期使用，不适合快速演示

作品集 MVP 需要 5-7 天能做出可演示版本。  
`cheat-on-content` 这种完整状态机适合做长期系统，不适合直接变成你的第一个大项目。

你应该学习它的最小骨架，而不是完整复刻。

---

## 9. 如何迁移到我的方向

### 9.1 迁移到 AI Workbench

可以把它抽象成一个通用工作流：

```text
task-score
  -> task-predict
  -> task-execute
  -> task-retro
  -> workflow-bump
```

对应关系：

| cheat-on-content | AI Workbench |
|---|---|
| script | 一个任务需求 |
| rubric | 任务质量评价规则 |
| prediction | 执行前的质量/风险预测 |
| publish | 执行任务 |
| retro | 任务完成后的结果复盘 |
| bump | 升级提示词、流程、检查清单 |

可做成：

```text
输入任务目标
  -> AI 判断任务难度、风险、预期产出
  -> 执行任务
  -> 用户标记结果是否满意
  -> AI 复盘失败点
  -> 更新下一次任务的 checklist
```

最适合你的 MVP 模块：

```text
任务前检查
任务后复盘
提示词版本升级记录
```

---

### 9.2 迁移到求职投递复盘

这是非常适合迁移的方向。

可以设计成：

```text
job-score
  -> apply-predict
  -> submit
  -> interview-retro
  -> resume-bump
```

对应关系：

| cheat-on-content | 求职投递 |
|---|---|
| 稿子 | JD + 简历 + 项目经历 |
| score | 岗位匹配打分 |
| predict | 投递前预测：是否值得投、回复概率、风险点 |
| publish | 实际投递 |
| retro | 记录是否已读、约面、拒信、面试反馈 |
| bump | 调整简历表达、项目描述、岗位筛选规则 |

可设计的文件：

```text
.job-state.json
resume_rules.md
application-memo.md
applications/
jd/
retros/
```

核心闭环：

```text
输入 JD
  -> AI 打分：技能匹配、项目相关度、行业匹配、风险点
  -> 投递前写预测
  -> 记录投递结果
  -> 复盘为什么没回复 / 为什么约面
  -> 更新简历和投递策略
```

这个方向比内容创作更贴合你的当前目标。

---

### 9.3 迁移到 AI 电商运营工作台

可以把它迁移成：

```text
content-score
  -> launch-predict
  -> publish
  -> ops-retro
  -> strategy-bump
```

对应关系：

| cheat-on-content | AI 电商运营工作台 |
|---|---|
| script | 商品笔记 / 短视频脚本 / 客服话术 |
| rubric | 内容/商品/客服评价规则 |
| prediction | 发布前预测表现 |
| publish | 发布小红书笔记、抖音短视频或商品文案 |
| retro | 记录曝光、点击、收藏、咨询、成交 |
| bump | 调整卖点权重、关键词策略、内容模板 |

结合你前面的小红书种草助手，可以做成：

```text
商品信息
  -> AI 生成笔记
  -> AI 评分：卖点清晰度、搜索关键词、种草感、合规风险、转化引导
  -> 发布前预测：自然流量/互动/咨询概率
  -> 发布后输入数据
  -> 复盘：标题问题、卖点问题、关键词问题、评论反馈
  -> 更新下一轮内容规则
```

第一版不要做自动抓数据。  
可以先手动输入：

```text
曝光
点赞
收藏
评论
私信
成交/咨询
用户评论关键词
```

然后输出：

```text
本轮种草表现
预测偏差
下轮标题建议
下轮卖点调整
是否值得继续投放
```

这就是把 `cheat-on-content` 的闭环迁移到电商运营。

---

## 10. 我学完它后可以写进作品集的表达

下面这些表达不能说“我开发了 cheat-on-content”，而是说“我拆解并迁移了它的设计思想”。

### 表达 1

系统拆解 `cheat-on-content` 的 score -> predict -> publish -> retro -> bump 工作流，理解其如何通过状态文件、不可变预测日志和复盘机制，把内容创作转化为可校准的实验系统。

### 表达 2

基于开源 skill 的 workflow 设计，抽象出“执行前判断 -> 执行记录 -> 结果回收 -> 规则升级”的通用 AI 工作台模式，并迁移到求职投递复盘和电商运营分析场景。

### 表达 3

分析其 `.cheat-state.json`、templates、hooks、adapters 的协作方式，学习如何在 AI Agent 项目中设计持久状态、流程约束和外部数据适配层。

### 表达 4

借鉴盲预测和 immutable log 机制，设计求职投递/电商内容发布前的预测记录与发布后复盘流程，避免 AI 分析被结果数据污染。

### 表达 5

将 `rubric_notes.md` 与 `rubric-memo.md` 的分离思想迁移到个人项目中，把“当前判断规则”和“历史复盘证据”分开管理，提升 AI 工作流的可解释性和可迭代性。

---

## 11. 对我下一步最实际的建议

不要直接复刻这个项目。

你应该先做一个更小的迁移版：

```text
AI 小红书种草运营工作台
  -> 内容发布前评分
  -> 发布前预测
  -> 发布后手动输入数据
  -> 自动复盘
  -> 更新下一轮内容规则
```

最小文件结构可以是：

```text
AI小红书种草运营工作台/
├── .ops-state.json
├── content_rules.md
├── content_memo.md
├── drafts/
├── predictions/
├── retros/
└── templates/
```

第一版只做 5 个动作：

```text
score-note
predict-note
publish-register
retro-note
bump-rules
```

这样既能学习 `cheat-on-content` 的系统思维，又不会陷入它完整工程复杂度。

