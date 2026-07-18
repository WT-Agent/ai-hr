# 网腾无限AI 企业HR面试问题

## 项目介绍

**网腾无限AI 企业HR面试问题** 是一款深受 nomads.com (Nomad List) 数据看板风格启发、面向企业 HR、面试官及招聘团队的互动面试套件微应用。该应用基于 Vue 3 + Vite + Vanilla CSS 打造，HR 能对招聘岗位的胜任力维度（专业技能、逻辑抗压、团队协作、潜能学习、稳定性）进行滑动评分并输入岗位核心要求。AI 会精算并呈现“HR期望 vs AI基准”的对比看板，并按选定流派（含大厂冰山模型、九巨擘HR圆桌等）生成包含胜任力矩阵、5 组行为提问（含 STAR 标准与防套话追问）及录用建议的面试套件。同时，页面内置了支持 Web Audio 盖章发声与 Offer Pass 累加的录用印章组件。

### 核心特性
- **胜任力五维评分交互**：专业技能匹配 (Skills)、逻辑思维与抗压 (Logic)、团队协作与沟通 (Teamwork)、潜能与学习力 (Potential)、稳定性与忠诚度 (Stability) 的无缝滑动打风。
- **Offer Pass 录用印章 (Offer Stamp)**：前端利用 Web Audio API 动态合成沉稳的机械盖章音效，点击印章即可累加 Offer Pass 计数并浮现“Offer Pass +1”渐隐动画。
- **AI 判定与胜任力对比看板**：自动提取 AI 回复末尾的共识数据，并在前端渲染并排的双轨数据对比图表。
- **HR 面试套件名片式历史记录**：将所有面试套件与录用印章计数持久化于本地，以精美的“面试套件卡”网格形式展示，支持详情回填、单条删除与一键清空。
- **右上角常驻分享**：磨砂玻璃悬浮分享按钮，快速呼出微信朋友圈分享指引。
- **并排二维码打赏与联系弹窗**：打赏栏并排展示微信/支付宝支付码，Contact Us 弹窗并排展示微信/钉钉联系码。
- **无下划线自适应弹窗**：条款及隐私弹窗支持 max-height 限高内滚动，而二维码弹窗高度自适应，防止双滚动条。

## 快速启动

### 1. 克隆项目
```bash
git clone https://github.com/WT-Agent/ai-hr.git
cd ai-hr
```

### 2. 安装依赖
项目强制使用 pnpm 作为包管理器：
```bash
pnpm install
```

### 3. 配置本地开发环境
复制并配置环境变量：
```bash
cp .env.example .env
```
在 `.env` 中填入您的 API 密钥：
- `DEEPSEEK_API_KEY`: 您的 DeepSeek 开发者 API 密钥（用于文本生成任务）

### 4. 开发与构建
启动本地开发服务（支持本地反向代理中转，防止前端密钥泄露）：
```bash
pnpm dev
```
打包生产静态资源（打包输出在 `dist/` 目录中，支持零成本部署于 GitHub Pages、Vercel 或 OSS 容器）：
```bash
pnpm build
```

## 联系我们

如果您在使用过程中有任何问题、建议或商务合作，可以通过微信或钉钉联系我们。

## 打赏支持

如果本项目对您有帮助，欢迎打赏作者喝杯咖啡。您的支持是项目持续优化与更新的动力。

| 微信支付 | 支付宝 |
| :---: | :---: |
| <img src="asset/tenpay.png" width="180" alt="微信支付" /> | <img src="asset/alipay.png" width="180" alt="支付宝" /> |

## 版权与许可

本项目基于 MIT License 开源协议。

Copyright (c) 2026. All rights reserved.
