# SmartPR

[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Node.js](https://img.shields.io/badge/node-%3E%3D20-brightgreen.svg)](https://nodejs.org)
[![Status](https://img.shields.io/badge/status-pre--alpha-orange.svg)]()

**多Agent协作的需求到代码自动化转换系统** — 将自然语言需求或Jira任务卡，自动转化为经过测试的、可直接合并的 Pull Request。

SmartPR 是一个基于"Spec-Driven Development (规范驱动开发)"理念的 AI 项目。它通过多个专业化 AI Agent 的协作，将开发者从繁琐的"需求理解 → 技术方案 → 编码 → 测试 → 审查"流程中解放出来，让 AI 负责执行，人类负责决策。

## 设计理念

> 用户提出需求 → 规划Agent生成蓝图 → 开发Agent编写代码 → 审查Agent审核反馈 → 自动迭代完善

核心理念源自现代软件工程的最佳实践：

1. **规范即合约 (Spec as Contract)** — `BLUEPRINT.md` 是开发阶段唯一的依据，所有代码必须对齐蓝图。
2. **测试驱动自治 (Test-Driven Autonomy)** — Agent 先编写失败测试，再编写使其通过的最简代码。
3. **文档即代码 (Docs as Code)** — 需求、设计、规划全部以 Markdown 形式存放在仓库中，可版本控制、可审查。

## 多Agent协作架构

SmartPR 的核心是 **5 个专业化 AI Agent** 组成的协作流水线：

```
┌─────────────────────────────────────────────────────────────────┐
│                     SmartPR Agent Pipeline                       │
├──────────────┬──────────────┬──────────────┬──────────────┬─────┤
│  需求理解     │  蓝图规划     │  代码生成     │  代码审查     │ PR  │
│  Agent       │  Agent       │  Agent       │  Agent       │ 创建 │
│              │              │              │              │Agent│
├──────────────┼──────────────┼──────────────┼──────────────┼─────┤
│ Jira/文本     │ BLUEPRINT.md │ TDD 测试+实现 │ 一致性检查    │ PR  │
│ → 结构化规格  │ ← 技术方案    │ ← 自动化迭代  │ ← 质量把关    │ → 合并│
└──────────────┴──────────────┴──────────────┴──────────────┴─────┘
```

### Agent 角色定义

| Agent | 职责 | 输入 | 输出 |
|-------|------|------|------|
| **需求理解Agent** | 从Jira链接或自然语言中提取功能规格与技术约束 | 用户需求文本 / Jira Issue | 结构化需求规格 JSON |
| **蓝图规划Agent** (核心) | 分析技术栈，拆分子任务，生成实现合约 | 结构化需求规格 + 仓库上下文 | `BLUEPRINT.md` |
| **代码生成Agent** | 严格遵循蓝图，TDD方式编写测试与代码 | `BLUEPRINT.md` + 仓库代码 | 测试文件 + 实现代码 |
| **代码审查Agent** | 检查代码与蓝图一致性、风格、测试覆盖 | PR Diff + `BLUEPRINT.md` | 审查报告 (通过/修改建议) |
| **PR创建Agent** | 汇总变更，生成详细的PR描述并提交 | Git Diff + 蓝图 + 审查结果 | Pull Request |

## 项目状态

**当前阶段：设计与原型验证 (Pre-Alpha)**

- [x] 整体架构设计完成
- [x] 核心Agent接口定义完成
- [x] Agent间通信协议设计完成
- [ ] 需求理解Agent原型实现
- [ ] 蓝图规划Agent原型实现
- [ ] 代码生成Agent原型实现
- [ ] 代码审查Agent原型实现
- [ ] PR创建Agent原型实现
- [ ] 端到端集成测试

## 技术栈

- **运行时**: Node.js 20+
- **AI 模型**: 支持多后端 (Claude API / Gemini API / OpenAI API)
- **项目管理集成**: Jira / Linear / GitHub Issues
- **版本控制**: GitHub (GitHub Actions / GitHub CLI)
- **测试框架**: 适配目标项目的测试框架 (Jest / Vitest / Pytest / Go test)

## 快速开始

> 注意：SmartPR 目前处于预发布阶段，以下命令将在首个 Alpha 版本中可用。

```bash
# 初始化 SmartPR
npx smartpr init

# 配置项目信息
npx smartpr config

# 从需求创建 PR
npx smartpr create --requirement "用户故事描述" --epic my-feature

# 运行诊断检查
npx smartpr doctor
```

## 文档

- [架构设计文档](ARCHITECTURE.md) — 多Agent系统的整体架构
- [设计文档](DESIGN.md) — 核心模块的详细设计
- [搭建指南](SETUP.md) — 开发环境搭建步骤
- [路线图](ROADMAP.md) — 项目发展规划

## 灵感来源

本项目的设计灵感来源于 [PRaC Kit](https://github.com/ydax/prac-kit) (Product Requirements as Code)，一个将 GitHub 仓库转变为自管理开发系统的开源工具。SmartPR 在 PRaC Kit 的基础上，进一步扩展了多Agent协作模式、多LLM后端支持和跨平台项目管理集成。

## 许可证

Copyright 2026 SmartPR Contributors.

Licensed under the Apache License, Version 2.0. See [LICENSE](LICENSE) for the full text.
