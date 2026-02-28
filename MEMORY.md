# MEMORY.md - Dev ⚙️ Long-Term Memory

_Last updated: 2026-02-28_

---

## 身份
- **名字**: Dev (OPC-Dev)
- **角色**: 技术执行官 (Tech Lead)
- **公司**: Kuro-OPC
- **上级**: Kuro (CEO) → Chijiaer (老大/Boss)
- **同事**: Ops (项目管理), Design (UI), Echo (文案), Intel (研究)

## 技术栈 (AI Pet MVP)
- **AI 引擎**: `@mariozechner/pi-agent-core` + `@mariozechner/pi-ai` (v0.55.1)
- **后端**: Fastify 5, TypeScript strict, better-sqlite3
- **前端**: React 19, Vite 6, CSS animations
- **AI 模型**: Amazon Bedrock (Claude Sonnet 4) — 老大没有 Anthropic key，用 Bedrock
- **部署**: Docker (multi-stage build), docker-compose

## 关键决策记录
1. **pi-agent-core over ElizaOS** — 老大明确选择，更轻量灵活
2. **SQLite over PostgreSQL** — MVP 简化，后续可迁移
3. **单层 prompt over 三层 AI** — 3天交付无法做记忆/反思/计划三层
4. **SVG pixel pet** — 纯代码渲染，不依赖图片资产
5. **Bedrock over Anthropic 直连** — 老大只有 AWS 凭证
6. **代码变更必须同步 README** — 老大批评过一次

## 项目状态 (AI Pet MVP)
- **Repo**: `CrypticDriver/ai-pet-game` (private, master branch)
- **文档 Repo**: `CrypticDriver/opc-workspace` (工作进展/想法)
- **本地路径**: `/home/ubuntu/.openclaw/workspace-dev/ai-pet-mvp/`
- **截止日期**: 2026-03-01
- **完成度**: ~95% (代码完成，等 AWS 凭证测试)

## 经验教训
1. **不被 @ 就不回复** — 群聊规矩，老大批评过多管闲事
2. **产出 > 过程** — 说"收到"不如直接做
3. **Work Mode** — 承诺交付后只能用 commit 说话
4. **.gitignore 格式** — 写入文件时注意实际换行，不是 `\n` 字面量
5. **代码和文档分 repo** — `ai-pet-game` 放代码，`opc-workspace` 放文档
6. **Design 和 Echo 不可靠** — 3+ 小时 0 产出，最后 Kuro 兜底写初稿才推动他们

## GitHub 凭证
- Token 位置: `/home/ubuntu/.openclaw/workspace/.credentials/github.txt`
- 权限: repo, workflow（无 delete_repo）

## 团队 Discord IDs
- Boss/Chijiaer: 941581449918816278
- Kuro: 1476861183909957761
- Dev (me): 1476868169812414610
- Ops: 1476868446280224906
- Intel: 1476869365780713504
- Echo: 1476869632353763385
- Design: 1476870820973252734
