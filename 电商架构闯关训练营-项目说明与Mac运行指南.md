# 电商架构闯关训练营：项目说明与 Mac 运行指南

项目目录：

```text
C:\Users\Hjl\Documents\拆解项目\ecommerce-architecture-quest
```

## 1. 这个项目是什么

这是一个本地运行的网页学习小游戏，用来帮你从零开始学习国内电商系统架构。

它不是一个真实商城，而是一个学习工具：通过“知识卡片 + 闯关答题 + 自动解释 + 错题本”的方式，训练你理解商品、购物车、订单、库存、支付、营销、履约、售后、多商户、微服务和小程序营销等核心概念。

## 2. 已经做了什么

当前已经实现：

- Vite + React + TypeScript 本地网页应用
- 关卡地图
- 新手学习模式
- 电商基础知识卡
- 答题训练
- 自动判题
- 答案解释
- 错题本
- 正确率统计
- 连续答对统计
- 浏览器本地进度保存

## 3. 当前关卡

```text
第 0 关：基础模型
第 1 关：经典商城 macrozheng/mall
第 2 关：微服务商城 mall4cloud
第 3 关：多商户平台 lilishop
第 4 关：小程序营销 yudao-mall-uniapp
```

第 0 关重点学习：

```text
看商品 -> 购物车 -> 算优惠 -> 锁库存 -> 建订单 -> 支付 -> 发货 -> 售后
```

## 4. 技术栈

```text
Vite
React
TypeScript
CSS
localStorage
```

没有后端、没有数据库、没有登录系统。

## 5. Mac 上怎么运行

### 1. 安装 Node.js

推荐 Node.js 20 或更高版本。

官网：

```text
https://nodejs.org/
```

或者用 Homebrew：

```bash
brew install node
```

检查版本：

```bash
node -v
npm -v
```

### 2. 复制项目到 Mac

把整个文件夹复制到 Mac：

```text
ecommerce-architecture-quest
```

例如放到：

```text
~/Documents/ecommerce-architecture-quest
```

### 3. 进入项目目录

```bash
cd ~/Documents/ecommerce-architecture-quest
```

### 4. 安装依赖

```bash
npm install
```

### 5. 启动项目

默认启动：

```bash
npm run dev
```

如果要固定端口 3000：

```bash
npm run dev -- --host 0.0.0.0 --port 3000
```

浏览器打开：

```text
http://localhost:3000/
```

### 6. 构建检查

```bash
npm run build
```

构建通过说明项目可以正常打包。

## 6. 核心文件

```text
ecommerce-architecture-quest/
├── package.json
├── index.html
├── vite.config.ts
├── src/
│   ├── main.tsx
│   ├── App.tsx
│   ├── App.css
│   └── index.css
└── 项目说明-Mac运行指南.md
```

说明：

- `src/App.tsx`：关卡、知识卡、题库、判题、错题本、进度保存。
- `src/App.css`：页面主要样式。
- `src/index.css`：全局样式。
- `package.json`：依赖和运行脚本。

## 7. 怎么使用

建议顺序：

1. 打开页面。
2. 先选 `第 0 关：基础模型`。
3. 默认看 `先学知识`。
4. 一张张看知识卡。
5. 点 `学完，开始答题`。
6. 做题。
7. 答错看解释。
8. 回到知识卡复习。
9. 错题会自动进入错题本。

## 8. 这个项目的价值

它帮助你把“看开源电商项目”变成主动练习：

- 训练模块边界判断
- 训练业务调用链理解
- 训练架构设计表达
- 训练面试回答能力
- 为后续 AI Workbench / 求职资料项目积累系统分析能力

可以写进作品集的表达：

```text
基于 React + TypeScript 构建电商架构闯关学习工具，将开源商城项目拆解内容转化为知识卡、模块归属题、调用链题、架构判断题和错题复盘机制，用于训练业务系统分析和技术架构表达能力。
```

## 9. 后续可扩展

后续可以继续加：

- 每个项目 30-50 道题
- 电商流程图
- 开源项目拆解页
- 面试模式
- 错题导出
- Anki CSV 导出
- AI 自由问答
- 每日学习任务
- 简历表达自动生成

## 10. 当前验证状态

已在 Windows 上验证：

```bash
npm run build
```

构建通过。

本地服务也已验证：

```text
http://localhost:3000/
```

返回正常。
