# Mac Codex 接手上下文：Windows 拆解项目

> 这份文档用于让 Mac 上的 Codex 直接接手，不要重新拆范围。  
> 当前 Windows 本地路径：`C:\Users\Hjl\Documents\拆解项目`  
> 当前日期：2026-06-17  
> 当前状态：Windows 端已经完成多轮项目拆解、市场调研、产品方向收敛和一份合规引流方案。下一步是让 Mac Codex 基于这些事实继续产出开发前文档和 MVP。

---

## 1. 这个拆解项目的一句话目标

帮助我从“不会完整开发项目的应届本科生”过渡到能用 Codex 做 AI 产品项目、开源项目拆解、市场调研、产品设计文档和求职作品集。

---

## 2. 这个项目到底在拆解什么

这个 Windows 工作区不是单一代码项目，而是一个“AI 产品学习与作品集准备工作区”。

它已经拆过和沉淀了四条主线：

```text
1. 开源 AI 信息聚合项目拆解
2. 电商 / AI 电商运营工具调研
3. 小红书种草运营助手产品方向
4. Codex 教学内容与合规引流方向
```

最核心的方向已经收敛为：

```text
AI 小红书种草运营工作台
```

不要再把范围扩回“大而全 AI Workbench”或“完整电商后台”。

---

## 3. 原始仓库 URL 与本地路径

### 3.1 finaldie/auto-news

- 原始仓库 URL：`https://github.com/finaldie/auto-news`
- Windows 本地路径：`C:\Users\Hjl\Documents\拆解项目\auto-news`
- 本地 Git remote：`https://github.com/finaldie/auto-news.git`
- 当前用途：作为成熟 AI 信息聚合 / LLM 流程拆解样本。
- 已产出分析文件：`auto-news-项目拆解报告.md`
- Mac Codex 不需要重新 clone 或重新深拆，除非用户明确要求二次复核。

### 3.2 XBuilderLAB/cheat-on-content

- 原始仓库 URL：`https://github.com/XBuilderLAB/cheat-on-content`
- Windows 临时只读路径：`C:\Users\Hjl\AppData\Local\Temp\cheat-on-content-readonly`
- 注意：Windows 端只做了静态阅读，没有安装、没有运行、没有改代码。
- 当前用途：学习 skill workflow、状态管理、盲预测、复盘和规则升级设计。
- 已产出分析文件：`cheat-on-content_skill深度拆解报告.md`
- Mac Codex 不要安装这个 skill，也不要把它直接用于内容创作；只学习它的 workflow 设计。

### 3.3 ecommerce-architecture-quest

- GitHub 仓库 URL：`https://github.com/792372962-cmd/ecommerce-architecture-quest`
- Windows 本地路径：`C:\Users\Hjl\Documents\拆解项目\ecommerce-architecture-quest`
- 本地 Git remote：`https://github.com/792372962-cmd/ecommerce-architecture-quest.git`
- 当前用途：此前做过的“电商架构闯关训练营”学习游戏。
- 状态：已经可运行、已打包 `ecommerce-architecture-quest-mac.zip`，并推送到 GitHub 私有仓库。
- 这不是当前下一步主线，只作为学习游戏和作品集补充。

### 3.4 当前 Windows 拆解项目仓库

- Windows 本地路径：`C:\Users\Hjl\Documents\拆解项目`
- 当前 Git 状态：根目录是一个 Git 仓库，之前没有 commit，没有 remote。
- 本次任务目标：把真实上下文整理成 Markdown 并同步到 GitHub，供 Mac Codex 接手。

---

## 4. 已经做到哪一步了

### 4.1 auto-news 开源项目拆解已完成

已经完成对 `finaldie/auto-news` 的 README、目录、配置和核心源码逻辑拆解。

重点理解：

```text
信息接入
  -> 清洗
  -> LLM 摘要/处理
  -> 输出
```

产物：

- `auto-news-项目拆解报告.md`

这个阶段的作用是学习成熟 AI 信息聚合项目的架构与 LLM 流程。

---

### 4.2 电商学习游戏已完成一版

此前做了一个 Vite + React + TypeScript 的学习游戏：

```text
电商架构闯关训练营
```

它用于帮助新手通过问答和关卡理解电商架构。  
但用户后来明确表示：现在更关心“AI 软件怎么提升电商运营效率”，不想先学传统电商源码。

相关产物：

- `ecommerce-architecture-quest/`
- `ecommerce-architecture-quest-mac.zip`
- `电商架构闯关训练营-项目说明与Mac运行指南.md`

当前不是主线，不要继续优先扩这个游戏。

---

### 4.3 AI 电商运营工作台方向已提出

先做了一版泛化的：

```text
AI 电商运营工作台
```

产物包括：

- `AI电商运营工作台-MVP方案.md`
- `AI电商运营工作台_产品流程图.md`
- `AI电商运营工作台_页面结构.md`
- `AI电商运营工作台_开发任务清单.md`
- `AI电商运营工作台_流程闭环与获利逻辑.md`
- `我的AI电商闭环起步方案.md`

但这个方向后来被进一步收窄：不要做完整电商后台，不要做登录、支付、订单、库存、数据库。

---

### 4.4 AI 电商工具市场调研已跑三轮

#### 第一轮

文件：

- `AI电商工具_市场调研工作流与第一轮样本.md`

主要做了：

```text
即创
有赞
微盟 WAI
妙手
店小秘
影刀
```

结论：AI 电商工具不是一个聊天框，而是嵌进内容、客服、投放、复盘、ERP/RPA 流程里。

#### 第二轮

文件：

- `AI电商工具_市场调研第二轮_平台闭环样本.md`

主要看平台闭环：

```text
千牛 / 淘宝商家 AI
阿里妈妈万相台
抖音电商罗盘 / 巨量千川
小红书聚光 / MIO
京东商智 / 京准通 / 言犀
聚水潭
快手磁力
拼多多商家工具
```

结论：最适合当前阶段的不是大而全后台，而是轻量 AI 经营助手。

#### 第三轮

文件：

- `AI电商工具_市场调研第三轮_小红书种草助手.md`

重点深挖：

```text
小红书聚光
MIO
蒲公英
千瓜数据
灰豚数据
新红 / 新榜
```

最终产品方向已明确：

```text
AI 小红书种草运营工作台
```

核心闭环：

```text
商品诊断
  -> 笔记生成
  -> 合规检查
  -> 关键词建议
  -> 投放/发布建议
  -> 复盘报告
```

---

### 4.5 cheat-on-content skill 拆解已完成

文件：

- `cheat-on-content_skill深度拆解报告.md`

重点学习它的：

```text
score
predict
publish
retro
bump
```

可迁移思想：

```text
执行前判断
  -> 执行记录
  -> 结果回收
  -> 复盘
  -> 规则升级
```

这套思想应该迁移到小红书种草工作台里：

```text
笔记发布前评分
  -> 发布前预测
  -> 发布后数据录入
  -> 复盘偏差
  -> 优化下一轮内容规则
```

---

### 4.6 Codex 教学 / 小红书引流方向已做合规判断

用户提出过：

```text
小红书发 Codex 或 VPN 教程
  -> 导流到闲鱼
  -> 交付服务
```

已整理为：

- `小红书Codex教学_合规引流方案.md`

明确结论：

```text
Codex / AI 项目教学可以做
VPN 教程和小红书导闲鱼交易不要做
```

推荐方向：

```text
应届生用 Codex 做 AI 项目 / 求职作品集
```

可做承接：

```text
Codex 新手项目陪跑包
AI 项目拆解模板
GitHub 上传指导
AI 求职作品集 7 天陪跑
```

---

## 5. 最重要的相关文件列表

Mac Codex 接手时优先读这些文件，按顺序：

```text
MAC_CODEX_HANDOFF.md
AI电商工具_市场调研第三轮_小红书种草助手.md
cheat-on-content_skill深度拆解报告.md
AI电商运营工作台-MVP方案.md
AI电商运营工作台_产品流程图.md
AI电商运营工作台_页面结构.md
AI电商运营工作台_开发任务清单.md
小红书Codex教学_合规引流方案.md
```

如果要了解前因，再读：

```text
AI电商工具_市场调研第二轮_平台闭环样本.md
AI电商工具_市场调研工作流与第一轮样本.md
AI电商运营工作台_流程闭环与获利逻辑.md
我的AI电商闭环起步方案.md
auto-news-项目拆解报告.md
电商架构闯关训练营-项目说明与Mac运行指南.md
```

---

## 6. 可以忽略的文件和目录

Mac Codex 不要优先处理这些：

```text
auto-news/
ecommerce-architecture-quest/
ecommerce-architecture-quest-mac.zip
*.docx
adjust_docx_font.py
adjust_docx_table_font.py
rebuild_docx_table_font.py
```

原因：

- `auto-news/` 是外部原始仓库 clone，已经完成拆解，不需要随当前上下文仓库同步。
- `ecommerce-architecture-quest/` 是单独 GitHub 仓库，已独立推送。
- `ecommerce-architecture-quest-mac.zip` 是打包文件，不需要进上下文仓库。
- `*.docx` 和几个 Python 脚本是论文/文档排版相关，不属于 AI 产品拆解主线。

---

## 7. 目前已知问题

### 7.1 根目录 Git 仓库此前没有 remote

Windows 根目录 `拆解项目` 是 Git 仓库，但此前：

```text
No commits yet on master
no remote
```

本次需要补 commit 和 remote 后才能让 Mac 端拉取。

### 7.2 方向容易发散

用户在探索中出现过多个方向：

```text
开源项目拆解
电商架构学习游戏
AI 电商运营工作台
小红书种草助手
Codex 教学引流
求职作品集
```

当前必须收敛，不要重新横向调研。

### 7.3 不要做 VPN 方向

VPN 教程、售卖、导流存在法律和平台合规风险。  
后续内容和产品设计都不要往 VPN 走。

### 7.4 不要做完整电商后台

明确不要加入：

```text
登录
支付
订单
库存
真实广告投放
真实平台授权
数据库
自动发布
```

当前阶段只做可演示的轻量工作台。

### 7.5 数据暂时手动输入

第一版不要接小红书 API 或爬虫。  
复盘数据先让用户手动填：

```text
曝光
点赞
收藏
评论
私信
成交/咨询
用户评论关键词
```

---

## 8. 希望 Mac Codex 下一步接着做什么

Mac Codex 不要重新拆范围，直接基于当前结论继续。

### 8.1 第一优先级：生成小红书种草工作台开发前文档

请直接生成 3 个 Markdown 文件：

```text
AI小红书种草运营工作台_产品流程图.md
AI小红书种草运营工作台_页面结构.md
AI小红书种草运营工作台_开发任务清单.md
```

要求：

- 只围绕“小红书种草助手”。
- 不扩展成多平台。
- 不加入登录、支付、订单、库存、数据库。
- 不做真实发布和真实投放。
- 保留 `cheat-on-content` 迁移来的复盘闭环思想。

核心模块固定为：

```text
商品种草诊断
笔记生成
合规与风险检查
关键词和投放建议
复盘报告
```

### 8.2 第二优先级：进入 MVP 开发

如果用户确认继续开发，做一个单页前端工作台。

建议页面结构：

```text
左侧：任务导航
中间：结构化输入
右侧：AI 输出结果
底部：复制 / 导出 Markdown / 保存本次结果
```

第一版可以先不接真实 AI API，用 mock 输出或本地 prompt 结构展示完整交互。

### 8.3 第三优先级：把作品集表达整理出来

开发或文档完成后，补：

```text
项目 README
作品集说明
简历项目描述
面试讲解稿
```

表达重点：

```text
基于国内电商平台调研
抽象小红书种草运营闭环
设计 AI 工作台
支持内容生成、合规检查、关键词建议、复盘报告
引入预测-复盘-规则升级思想
```

---

## 9. Mac Codex 不要做什么

不要做：

```text
不要重新调研 auto-news
不要重新安装 cheat-on-content
不要继续扩展电商架构小游戏
不要做 VPN 教程
不要做小红书导闲鱼交易方案
不要把产品扩成完整电商 ERP
不要接真实订单/库存/支付/数据库
不要接真实小红书登录或自动发布
```

当前目标不是“找更多方向”，而是把已经收敛的方向做成可展示 MVP。

---

## 10. 最短接手指令

Mac Codex 可以直接执行：

```text
读取 MAC_CODEX_HANDOFF.md 和 AI电商工具_市场调研第三轮_小红书种草助手.md。
不要重新拆范围。
直接生成：
1. AI小红书种草运营工作台_产品流程图.md
2. AI小红书种草运营工作台_页面结构.md
3. AI小红书种草运营工作台_开发任务清单.md

目标是为后续开发单页 MVP 做准备。
```

