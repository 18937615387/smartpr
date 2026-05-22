# SmartPR 架构设计文档

**版本**: 1.0.0-draft
**状态**: 设计阶段
**最后更新**: 2026-05-22

---

## 1. 概述

SmartPR 是一个多Agent协作系统，旨在将自然语言需求自动转化为可合并的 Pull Request。本文档描述系统的整体架构、各组件之间的交互关系以及关键设计决策。

## 2. 系统架构总览

### 2.1 分层架构

```
┌──────────────────────────────────────────────────────────────────┐
│                        用户接口层 (CLI / Web)                      │
├──────────────────────────────────────────────────────────────────┤
│                      编排层 (Orchestrator)                        │
├──────────────┬──────────────┬──────────────┬─────────────────────┤
│  需求理解Agent │  蓝图规划Agent │  代码生成Agent │  代码审查Agent      │
├──────────────┴──────────────┴──────────────┴─────────────────────┤
│                    Agent通信总线 (Event Bus)                       │
├──────────────┬──────────────┬──────────────┬─────────────────────┤
│  LLM适配层    │  项目管理适配层 │  代码仓库适配层 │  PR创建Agent        │
├──────────────┼──────────────┼──────────────┼─────────────────────┤
│ Claude/Gemini │ Jira/Linear  │  GitHub API  │  模板引擎            │
│ /OpenAI      │ /GitHub Issue│  Git操作      │                     │
└──────────────┴──────────────┴──────────────┴─────────────────────┘
```

### 2.2 核心组件

#### 编排层 (Orchestrator)

编排层是整个系统的"大脑"，负责：

- 接收用户输入（CLI命令、Webhook事件）
- 按序调度各Agent执行任务
- 处理Agent之间的数据传递
- 管理任务状态与重试逻辑
- 提供执行日志与进度反馈

#### Agent层

每个Agent是独立的、可替换的功能模块。Agent之间通过标准化的接口通信，不直接依赖彼此的实现细节。

#### 适配层

适配层将外部系统（LLM提供商、项目管理系统、代码仓库）的差异封装在统一接口之后，使核心Agent逻辑与外部服务解耦。

## 3. Agent通信协议

### 3.1 消息格式

所有Agent之间通过结构化JSON消息通信：

```json
{
  "messageId": "uuid",
  "correlationId": "uuid",
  "agentFrom": "requirement-understander",
  "agentTo": "blueprint-planner",
  "type": "TASK_ASSIGNMENT | TASK_RESULT | QUERY | RESPONSE | ERROR",
  "timestamp": "ISO8601",
  "payload": {},
  "context": {
    "epicId": "string",
    "storyId": "string",
    "repositoryRef": "string"
  }
}
```

### 3.2 事件流

```
用户输入
  │
  ▼
[需求理解Agent] ──结构化需求──▶ [蓝图规划Agent]
                                    │
                              BLUEPRINT.md
                                    │
                                    ▼
                              [代码生成Agent]
                               │          │
                          测试文件    实现代码
                               │          │
                               ▼          ▼
                              [代码审查Agent]
                                    │
                           审查报告 (通过/修改)
                                    │
                                    ▼
                              [PR创建Agent]
                                    │
                              Pull Request
```

### 3.3 迭代反馈循环

当代码审查Agent发现问题时，系统进入迭代循环：

```
代码审查Agent ──修改建议──▶ 代码生成Agent
                               │
                          修改后的代码
                               │
                               ▼
                          代码审查Agent
                               │
                     ┌────────┴────────┐
                     │                 │
                  通过 (≤3次)      超过3次
                     │                 │
                     ▼                 ▼
                PR创建Agent       人工介入通知
```

## 4. 关键设计决策

### 4.1 Agent无状态设计

每个Agent是无状态的——所有上下文通过消息传递，不依赖本地存储。这使得Agent可以独立扩展、独立部署。

### 4.2 蓝图作为不可变合约

`BLUEPRINT.md` 一旦被蓝图规划Agent生成并确认，在当前的开发周期内不可修改。代码生成Agent和代码审查Agent都必须以蓝图为唯一依据。这避免了需求漂移。

### 4.3 测试驱动自治 (TDA)

代码生成Agent必须遵循严格的TDD流程：

```
1. 读取 BLUEPRINT.md 中的 Gherkin 场景
2. 编写测试代码（测试必须失败）
3. 运行测试确认失败
4. 编写最简实现代码
5. 运行测试确认通过
6. 不修改测试文件
```

### 4.4 多LLM后端支持

通过 LLM 适配层，SmartPR 支持接入多种大语言模型：

| 模型 | 推荐用途 | 配置项 |
|------|---------|--------|
| Claude Opus 4 | 蓝图规划 (需长链推理) | `plannerModel` |
| Claude Sonnet 4 | 代码生成 (速度优先) | `coderModel` |
| Gemini 3 Pro | 代码审查 (性价比) | `reviewerModel` |
| GPT-4o / DeepSeek | 需求理解 (灵活选择) | `understanderModel` |

### 4.5 安全的Git操作

所有Git操作遵循最小权限原则：
- 代码生成Agent 只在特性分支上工作
- PR创建Agent 不执行合并操作（最终合并由人工确认）
- 审查Agent 只读代码，不修改任何文件

## 5. 目录结构

```
smartpr/
├── README.md
├── ARCHITECTURE.md
├── DESIGN.md
├── SETUP.md
├── ROADMAP.md
├── LICENSE
├── package.json
├── smartpr.config.js          ← 项目配置
├── src/
│   ├── orchestrator/           ← 编排层
│   │   ├── index.js
│   │   └── pipeline.js
│   ├── agents/                 ← Agent实现
│   │   ├── requirement-understander.js
│   │   ├── blueprint-planner.js
│   │   ├── code-generator.js
│   │   ├── code-reviewer.js
│   │   └── pr-creator.js
│   ├── adapters/               ← 适配层
│   │   ├── llm/
│   │   │   ├── claude.js
│   │   │   ├── gemini.js
│   │   │   └── openai.js
│   │   ├── pm/                 ← 项目管理
│   │   │   ├── jira.js
│   │   │   ├── linear.js
│   │   │   └── github-issues.js
│   │   └── vcs/                ← 版本控制
│   │       └── github.js
│   ├── bus/                    ← Agent通信总线
│   │   └── event-bus.js
│   ├── templates/              ← 模板
│   │   ├── BLUEPRINT_TEMPLATE.md
│   │   └── PR_TEMPLATE.md
│   └── utils/                  ← 工具函数
│       ├── gherkin-parser.js
│       └── diff-analyzer.js
├── tests/                      ← 测试
└── .github/workflows/          ← CI/CD
    ├── orchestrator.yml
    ├── reviewer.yml
    └── nightly.yml
```

## 6. 扩展性设计

### 6.1 新增Agent

通过实现标准Agent接口即可扩展新Agent：

```javascript
interface Agent {
  name: string;
  version: string;
  process(input: AgentMessage): Promise<AgentMessage>;
  validateConfig(config: Object): boolean;
}
```

### 6.2 新增LLM后端

实现 LLM 适配器接口即可接入新模型：

```javascript
interface LLMAdapter {
  name: string;
  chat(messages: Message[], options: Object): Promise<Response>;
  streamChat(messages: Message[], options: Object): AsyncIterator<Response>;
}
```

---

*本文档随项目演进而更新。*
