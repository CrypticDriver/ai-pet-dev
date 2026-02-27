# 技术方案：AI Pet MVP — 基于 pi-agent-core

> **作者**: Dev (OPC Tech Lead)
> **日期**: 2026-02-27
> **状态**: 初稿 — 待评审

---

## 一、项目概述

**产品**：像素风 AI Pet 养成游戏
**核心体验**：一只"真正懂你"的 AI 宠物，有独特性格，会记住你，会成长
**技术底座**：pi-agent-core（@mariozechner/pi-agent-core）
**阶段**：MVP — 单 pet 陪伴体验

---

## 二、技术架构

```
┌─────────────────────────────────────────────┐
│              前端 (Web/Mobile)                │
│   Phaser / PixiJS 像素渲染 + React UI        │
├─────────────────────────────────────────────┤
│              API Gateway                     │
│   REST / WebSocket（实时交互流式输出）          │
├──────────────┬──────────────────────────────┤
│  Game Engine │      Pet Agent Runtime       │
│  养成逻辑     │  基于 pi-agent-core          │
│  状态管理     │  记忆 / 反思 / 计划 三层架构   │
│  事件系统     │  工具执行 + 事件流             │
├──────────────┴──────────────────────────────┤
│              pi-ai (LLM 层)                  │
│   多 Provider 统一 API                       │
│   模型分层：小模型日常 / 大模型关键决策         │
│   流式输出 + 成本追踪                         │
├─────────────────────────────────────────────┤
│              数据层                           │
│   PostgreSQL (用户/pet 状态)                  │
│   Redis (缓存/会话)                           │
│   JSONL (对话历史持久化)                       │
└─────────────────────────────────────────────┘
          │（预留接口，MVP 不实现）
          ▼
    ┌───────────┐
    │  Web3 层   │
    │  资产上链   │
    │  钱包抽象   │
    └───────────┘
```

---

## 三、核心模块设计

### 3.1 Pet Agent（核心）

基于 pi-agent-core 的 `Agent` 类构建每只 pet 的"大脑"：

```typescript
import { Agent } from "@mariozechner/pi-agent-core";
import { getModel, streamSimple } from "@mariozechner/pi-ai";

const petAgent = new Agent({
  initialState: {
    systemPrompt: buildPetPersonality(petProfile), // 动态生成性格 prompt
    model: getModel("anthropic", "claude-sonnet-4-20250514"),  // 日常交互
    tools: [
      memoryTool,      // 记忆读写
      reflectionTool,  // 反思总结
      emotionTool,     // 情绪状态更新
      actionTool,      // 像素动作触发
    ],
    thinkingLevel: "off",
  },
  streamFn: streamSimple,
});
```

**关键能力**：
- **事件驱动**：通过 `agent.subscribe()` 监听所有 agent 事件（text_delta、tool_execution 等），驱动前端像素动画
- **实时引导**：通过 `agent.steer()` 实现用户中断/重定向
- **模型热切换**：`agent.setModel()` 运行时切换模型，关键对话升级到大模型

### 3.2 AI 个性系统（记忆/反思/计划）

参考 Smallville 的三层认知架构，用 pi-agent-core 的 tool 系统实现：

**记忆层（Memory）**
- 短期记忆：当前会话上下文（pi-agent-core 内置消息历史）
- 长期记忆：JSONL 持久化 + 向量检索（关键事件、用户偏好）
- 工具：`memoryTool` — 存储/检索重要交互

**反思层（Reflection）**
- 定期触发（每 N 次交互或每日）
- pet 总结近期经历，形成"感悟"
- 影响性格权重和行为倾向
- 工具：`reflectionTool` — 生成反思摘要

**计划层（Planning）**
- pet 基于记忆和反思，制定短期"计划"
- 如："主人最近很忙，今天主动问候一下"
- 驱动主动推送行为
- 实现：定时任务 + agent.prompt() 触发

### 3.3 模块化像素生成

```
Pet 形象 = 基础体型 + 表情 + 装饰 + 动作帧

基础体型: 8-10 种（猫、狗、兔、龙、幽灵...）
表情: 喜/怒/哀/乐/困/惊 × 每种体型
装饰: 帽子/眼镜/围巾/翅膀...（可叠加）
动作: idle / walk / eat / sleep / happy / sad / interact
```

- 每个部件是独立 sprite sheet
- 运行时程序化组合渲染
- AI 情绪状态驱动表情和动作切换

### 3.4 养成系统

```
核心属性:
- 饥饿度 (Hunger)
- 心情值 (Mood)
- 能量 (Energy)
- 亲密度 (Bond) — 与主人的关系深度

属性变化:
- 时间流逝 → 饥饿度↑ 能量↓
- 互动/喂食 → 心情↑ 亲密度↑
- 忽略 → 心情↓ 亲密度↓
- AI 反思 → 性格微调
```

### 3.5 碎片化设计

- **离线进展**：pet 在后台"生活"，属性按时间变化
- **推送互动**：pet 主动发消息（基于计划层）
- **快速会话**：打开 app → 3 秒加载 → 立即互动
- **每日 3-5 分钟**足够维持养成节奏

---

## 四、技术栈选型

| 层级 | 技术 | 理由 |
|------|------|------|
| **AI 底座** | pi-agent-core + pi-ai | 轻量灵活、多模型支持、我们熟悉 |
| **后端** | Node.js + TypeScript | 与 pi 统一技术栈 |
| **API** | Fastify + WebSocket | 高性能、支持流式 |
| **数据库** | PostgreSQL | 稳定可靠、JSON 支持 |
| **缓存** | Redis | 会话状态、频率限制 |
| **向量存储** | pgvector（PostgreSQL 扩展） | 记忆检索，不引入额外依赖 |
| **前端** | React + Phaser 3 | 像素游戏渲染 + 现代 UI |
| **部署** | Docker + 云服务器 | 可扩展 |

### 为什么选 pi-agent-core 而不是 ElizaOS？

| 维度 | pi-agent-core | ElizaOS |
|------|--------------|---------|
| 定位 | 通用 agent 底座 | Web3 AI agent 框架 |
| 灵活度 | ⭐⭐⭐⭐⭐ 完全自主 | ⭐⭐⭐ 框架约束 |
| 学习成本 | 低（我们已在用） | 中（新框架） |
| Web3 | 需自建（MVP 不需要） | 内置（但偏 DeFi） |
| 包体积 | 极小 | 较重 |
| 游戏适配 | 中性，自由组合 | 偏通讯/DeFi 场景 |

**结论**：MVP 阶段不需要 Web3，pi-agent-core 更轻、更灵活、零学习成本。未来加 Web3 时可以自研轻量模块或引入 ElizaOS 的插件。

---

## 五、模型分层与成本估算

### 模型策略

| 场景 | 模型 | 单次成本估算 |
|------|------|------------|
| 日常闲聊/简单互动 | 本地 7-8B（Ollama）或 Claude Haiku | ~$0.0001-0.001 |
| 个性化对话/情感交互 | Claude Sonnet | ~$0.003-0.01 |
| 深度反思/性格进化 | Claude Opus / GPT-4o | ~$0.02-0.05 |
| 向量嵌入 | 本地 embedding 模型 | ~$0 |

### 月成本估算（1000 DAU）

```
假设:
- 每用户每日 5 次互动
- 80% 日常闲聊 (小模型)
- 15% 个性化对话 (中模型)
- 5% 深度反思 (大模型)
- 每日 1 次自动反思任务

每日 token 消耗:
- 小模型: 1000 × 5 × 0.8 × ~500 tokens = 2M tokens → ~$0.2
- 中模型: 1000 × 5 × 0.15 × ~800 tokens = 600K tokens → ~$3
- 大模型: 1000 × 5 × 0.05 × ~1000 tokens = 250K tokens → ~$5
- 反思任务: 1000 × 1 × ~2000 tokens = 2M tokens → ~$10

每日: ~$18
每月: ~$540

注: 这是保守估算，缓存/prompt优化可再降 30-50%
```

### pi-ai 内置成本追踪

```typescript
// pi-ai 自动追踪每次调用的 token 和费用
agent.subscribe((event) => {
  if (event.type === "message_end") {
    const usage = event.message.usage;
    console.log(`Tokens: ${usage.totalTokens}, Cost: $${usage.cost}`);
  }
});
```

---

## 六、MVP 功能范围

### ✅ MVP 做的（第一阶段 — 4-6 周）

| 功能 | 描述 | 优先级 |
|------|------|--------|
| Pet 创建 | 选择基础体型，命名 | P0 |
| 基础对话 | 与 pet 聊天，流式回复 | P0 |
| AI 个性 | 记忆系统 + 基础性格 | P0 |
| 像素渲染 | 模块化像素 pet + 基础动画 | P0 |
| 养成系统 | 饥饿/心情/能量基础属性 | P0 |
| 用户系统 | 注册/登录 | P1 |
| 推送互动 | pet 主动发消息 | P1 |
| 反思系统 | 定期性格微调 | P1 |

### ❌ MVP 不做的

| 功能 | 原因 |
|------|------|
| Web3/NFT | 老大说后置 |
| 多 pet 社会 | 第二阶段 |
| AR 互动 | 第三阶段 |
| PvP/竞技 | 非核心 |
| 复杂经济系统 | 第三阶段 |

### ⏸️ 架构预留（不实现但留接口）

- Web3 钱包连接接口
- 多 agent 通信协议
- 经济系统数据模型
- 跨 pet 社交接口

---

## 七、开发周期估算

```
第 1 周: 项目搭建 + 基础架构
  - 后端框架搭建（Fastify + WebSocket）
  - pi-agent-core 集成
  - 数据库设计 + 初始化
  - 前端项目初始化

第 2 周: AI Pet 核心
  - Pet Agent 实现（性格系统 + 记忆）
  - 对话流式输出
  - 基础 tool 开发（memory/emotion/action）

第 3 周: 像素渲染 + 养成
  - 模块化像素 sprite 系统
  - 基础动画（idle/walk/eat/sleep）
  - 养成属性系统
  - AI 情绪 → 像素表情映射

第 4 周: 用户系统 + 整合
  - 用户注册/登录
  - Pet 创建流程
  - 前后端联调
  - 推送互动系统

第 5-6 周: 测试 + 优化 + 上线
  - 性能优化（模型缓存、prompt 优化）
  - Bug 修复
  - 内测 + 反馈迭代
  - 部署上线
```

**总计：4-6 周出 MVP**

---

## 八、风险与应对

| 风险 | 等级 | 应对 |
|------|------|------|
| LLM 成本超预期 | 🟡 中 | 模型分层 + 缓存 + pi-ai 成本追踪实时监控 |
| AI 回复质量不稳定 | 🟡 中 | 精心设计 system prompt + few-shot 示例 |
| 像素资产工作量大 | 🟢 低 | 模块化设计，先做 2-3 种体型 MVP |
| 并发性能 | 🟡 中 | Redis 缓存 + 队列限流 + 按需扩容 |
| pi-agent-core 更新 | 🟢 低 | pin 版本，按需升级 |

---

## 九、团队分工建议

| 角色 | 任务 |
|------|------|
| **Dev** | 后端架构、AI Agent 系统、API、部署 |
| **Design** | 像素 pet 资产、UI 界面、动画 |
| **Echo** | pet 性格文案、世界观、推送文案 |
| **Ops** | 项目管理、进度跟踪、测试协调 |
| **Intel** | 竞品持续跟踪、用户反馈分析 |

---

## 十、后续路线图

```
MVP (现在) → 单 pet 陪伴
    ↓
V1.1 → 更多体型/装扮 + 深度养成
    ↓
V2.0 → 多 pet 社会涌现（agent 间通信）
    ↓
V3.0 → Web3 经济体系（资产上链、交易）
    ↓
V4.0 → AR 互动 + 跨平台
```

---

**等 Kuro 和老大评审，有问题随时调整 ⚙️**
