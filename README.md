# 任务控制中心（Mission Control）

**AI 智能体编排开源面板。**

管理 AI 智能体舰队、分发任务、追踪成本、协调多智能体工作流 —— 自托管、零外部依赖、SQLite 驱动。

[![MIT 许可证](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Next.js 16](https://img.shields.io/badge/Next.js-16-black?logo=next.js)](https://nextjs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.7-3178C6?logo=typescript&logoColor=white)](https://typescriptlang.org/)
[![测试](https://img.shields.io/badge/Tests-577%20(282%20unit%20%2B%20295%20E2E)-brightgreen)](https://github.com/Naexen/mission-control)

---

> **Alpha 软件** — 任务控制中心正在活跃开发中。各版本间的 API、数据库结构、配置格式可能会发生变化。部署到生产环境前请查阅 [安全注意事项](#安全)。

## 目录

- [快速开始](#快速开始)
- [为什么团队选择任务控制中心](#为什么团队选择任务控制中心)
- [使用场景](#使用场景)
- [智能体入门](#智能体入门)
- [功能特性](#功能特性)
- [架构](#架构)
- [开发](#开发)
- [安全](#安全)
- [许可证](#许可证)

---

<table>
<tr><td><b>32 个面板</b></td><td>任务、智能体、技能、日志、Token、记忆、安全、定时任务、警报、Webhooks、流水线等 —— 全部在一个 SPA 界面中。</td></tr>
<tr><td><b>全实时更新</b></td><td>WebSocket + SSE 推送，智能轮询（离开时自动暂停）。数据永不过期。</td></tr>
<tr><td><b>零外部依赖</b></td><td>SQLite 数据库，一行命令运行。无需 Redis、Postgres、Docker。</td></tr>
<tr><td><b>角色权限控制</b></td><td>Viewer、Operator、Admin 三种角色，支持会话 + API Key 认证，以及 Google 登录（需管理员审批）。</td></tr>
<tr><td><b>质量门禁</b></td><td>内置 Aegis 审查系统，任务完成前必须通过签核。</td></tr>
<tr><td><b>技能中心</b></td><td>浏览、安装、 安全扫描来自 ClawdHub 和 skills.sh 仓库的智能体技能，支持磁盘 ↔ 数据库双向同步。</td></tr>
<tr><td><b>多网关</b></td><td>同时连接多个智能体网关。支持 OpenClaw、CrewAI、LangGraph、AutoGen、Claude SDK 等框架适配器。</td></tr>
<tr><td><b>循环任务</b></td><td>自然语言调度（"每天早上9点"），基于 Cron 的模板化任务生成。</td></tr>
<tr><td><b>Claude Code 桥接</b></td><td>只读集成，在面板中展示 Claude Code 团队任务、会话和配置。</td></tr>
<tr><td><b>智能体评估与安全</b></td><td>四层评估框架、信任评分、密钥检测、MCP 调用审计，以及安全策略配置（minimal/standard/strict）。</td></tr>
</table>

---

## 快速开始

### 一键安装

```bash
git clone https://github.com/Naexen/mission-control.git
cd mission-control
bash install.sh --local     # Linux/macOS
# Windows: PowerShell 中运行 .\install.ps1 -Mode local
```

安装完成后：

```bash
open http://localhost:3000/setup    # 创建管理员账号
```

安装脚本自动处理 Node.js 22+、pnpm、依赖项，并生成安全凭据。

### 手动安装

```bash
git clone https://github.com/Naexen/mission-control.git
cd mission-control
nvm use 22 && pnpm install
pnpm dev                    # http://localhost:3000/setup
```

### Docker 零配置

```bash
docker compose up           # 自动生成凭据，重启后数据持久化
```

### 生产环境加固

```bash
docker compose -f docker-compose.yml -f docker-compose.hardened.yml up -d
```

---

## 为什么团队选择任务控制中心

- **可预测的编排**：一个面板管理任务流、分发、质量门禁和审计追踪。
- **更快的响应**：实时查看智能体/任务/安全遥测，无需拼接多个工具。
- **本地优先部署**：基于 SQLite，无需 Redis/Postgres 等强制外部依赖。
- **安全默认开启**：RBAC、信任评分、密钥检测、生产加固部署配置。

## 使用场景

### 5 分钟搭建本地控制中心

```bash
bash install.sh --local
# 打开 /setup 创建管理员账号
# 从界面创建第一个智能体和任务
```

### 零编程定时任务

在任务面板中用自然语言描述调度规则，例如"每个工作日上午9点抓取竞品动态"，系统自动转换为 Cron 表达式并生成任务模板。

### 多智能体协作

通过智能体编队功能同步多个智能体状态，监控心跳，从单一面板启动/唤醒/休眠智能体。

---

## 智能体入门

### MCP 服务器（推荐）

```bash
# 添加到任意 Claude Code 智能体：
claude mcp add mission-control -- node /path/to/mission-control/scripts/mc-mcp-server.cjs

# 环境变量配置：
MC_URL=http://127.0.0.1:MC_API_KEY=<your-key>
```

35 个工具：agents、tasks、sessions、memory、soul、comments、tokens、skills、cron、status 等。

### CLI

```bash
pnpm mc agents list --json
pnpm mc tasks queue --agent Aegis --max-capacity 2 --json
pnpm mc events watch --types agent,task
```

### REST API

OpenAPI 规范文档在 `/docs`（运行时）。

---

## 功能特性

### 智能体管理
注册、配置、监控智能体。查看实时状态、心跳、会话、工具配置、模型链和 SOUL 内容。

### 任务看板
看板视图管理任务生命周期，支持循环任务、质量审查、Aegis 签核流程、优先级和标签。

### 记忆知识图谱
向量嵌入 + 知识图谱双引擎，支持语义搜索、链接追踪、MOC（知识地图）生成。

### 技能中心
从 ClawdHub 安装技能，支持磁盘 ↔ 数据库同步，内置安全扫描。

### 成本追踪
Token 消耗分析、按智能体/模型分项统计、性能洞察与优化建议。

### 安全审计
四层评估框架、信任评分、密钥泄露检测、MCP 工具调用审计、入侵尝试监控。

### 循环任务
自然语言调度，界面化 Cron 规则配置与执行历史。

### 活动流
实时事件 feed，支持按类型/智能体/时间范围过滤。

### 集成
支持 GitHub Issues 双向同步、Webhooks、GNAP 认证、OpenClaw 网关直连。

### 多网关
同时管理多个智能体网关，支持框架适配器（OpenClaw / CrewAI / LangGraph / AutoGen / Claude SDK）。

---

## 架构

```
src/app/          Next.js 页面 + API 路由（App Router）
src/components/   UI 面板和共享组件
src/lib/          核心逻辑、数据库、工具函数
.data/            SQLite 数据库 + 运行时状态（gitignored）
scripts/          安装、部署、诊断脚本
docs/             文档和指南
```

路径别名：`@/*` 映射到 `./src/*`

数据目录：`MISSION_CONTROL_DATA_DIR`（默认为 `.data/`）  
数据库路径：`<MISSION_CONTROL_DATA_DIR>/mission-control.db`

### 网关模式（可选）

设置 `NEXT_PUBLIC_GATEWAY_OPTIONAL=true` 可在无网关连接时独立运行。

---

## 开发

```bash
pnpm install
pnpm build
pnpm dev              # 开发模式 (localhost:3000)
pnpm start            # 生产模式
node .next/standalone/server.js   # 独立模式（构建后）
```

```bash
pnpm test             # 单元测试（vitest）
pnpm test:e2e         # 端到端测试（playwright）
pnpm typecheck        # tsc --noEmit
pnpm lint             # eslint
pnpm test:all         # 完整检查：lint + typecheck + test + build + e2e
```

**包管理器**：仅 pnpm（禁止 npm/yarn）  
**Node.js**：>= 22（推荐 LTS，24.x 亦支持）

---

## 安全

- **独立部署优先**：SQLite 本地存储，无需外部数据库依赖
- **强制身份认证**：所有访问需要登录，会话 + API Key 双认证
- **RBAC 权限体系**：Viewer / Operator / Admin 三级角色
- **审计日志**：记录所有操作，支持导出审计轨迹
- **安全评估**：内置信任评分、密钥泄露检测、注入攻击防护
- **加固部署**：`docker-compose.hardened.yml` 提供只读文件系统、能力降权、HSTS、网络隔离

---

## 许可证

MIT
