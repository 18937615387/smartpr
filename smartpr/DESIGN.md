# SmartPR 设计文档

**版本**: 1.0.0-draft
**状态**: 设计阶段
**最后更新**: 2026-05-22

---

## 1. 需求理解Agent 设计

### 1.1 目标

将非结构化的用户输入（自然语言、Jira链接）转化为结构化的功能规格。

### 1.2 处理流程

```
用户输入 (文本/Jira URL)
    │
    ▼
输入预处理 ── 如果是URL，调用项目管理API获取内容
    │
    ▼
LLM 分析 ── 提取：功能描述、验收条件、技术约束、影响范围
    │
    ▼
结构化输出 ── JSON Schema 格式的需求规格
```

### 1.3 输出格式

```json
{
  "title": "用户登录功能",
  "description": "实现基于JWT的用户登录接口",
  "acceptanceCriteria": [
    "Given 用户提供有效的用户名和密码 When 调用登录接口 Then 返回JWT token",
    "Given 用户提供无效凭据 When 调用登录接口 Then 返回401错误"
  ],
  "technicalConstraints": [
    "使用bcrypt进行密码哈希",
    "JWT过期时间设置为24小时"
  ],
  "affectedModules": ["auth", "user", "middleware"],
  "estimatedComplexity": "medium",
  "dependencies": ["user-service", "redis"]
}
```

### 1.4 关键设计点

- **输入验证**: 确保输入不是恶意prompt注入
- **上下文收集**: 自动扫描仓库结构，为LLM提供代码上下文
- **多源合并**: 支持同时从Jira和用户描述中提取信息并合并

---

## 2. 蓝图规划Agent 设计

### 2.1 目标

将结构化需求转化为详细的、可执行的实现方案 (`BLUEPRINT.md`)。

### 2.2 处理流程

```
结构化需求规格
    │
    ▼
技术栈分析 ── 扫描仓库: package.json/requirements.txt/go.mod 等
    │
    ▼
子任务拆分 ── LLM长链推理: 将需求拆解为原子性子任务
    │
    ▼
Gherkin生成 ── 为每个子任务生成 Given/When/Then 场景
    │
    ▼
文件级规划 ── 指定每个子任务需创建/修改的具体文件
    │
    ▼
BLUEPRINT.md ── 完整的实现合约文档
```

### 2.3 BLUEPRINT.md 结构

```markdown
# Blueprint: [功能名称]

## 元数据
- **需求引用**: REQ-001
- **复杂度**: medium
- **预估影响文件**: 5
- **生成时间**: 2026-05-22T10:00:00Z

## 技术约束
- 语言/框架: Node.js / Express
- 数据库: PostgreSQL
- 必须保持向后兼容

## 子任务

### 1. 创建用户模型
- **文件**: `src/models/user.js`
- **操作**: 新建
- **Gherkin**:
  - Given 数据库连接正常 When 创建新用户记录 Then 返回用户ID
  - Given 邮箱已存在 When 创建用户 Then 返回冲突错误

### 2. 实现登录接口
- **文件**: `src/routes/auth.js`
- **操作**: 修改
- **Gherkin**:
  - Given 有效凭据 When POST /api/auth/login Then 返回200和JWT
  - Given 无效凭据 When POST /api/auth/login Then 返回401

...

## 测试策略
- 单元测试: Jest
- 集成测试: Supertest
- 覆盖率目标: 80%
```

### 2.4 关键设计点

- **长链推理**: 蓝图生成是系统中最复杂的推理任务，需要分析技术栈、代码库结构、依赖关系等多个维度
- **子任务依赖图**: 自动识别子任务之间的依赖关系，生成DAG (有向无环图)
- **人类可审查**: 蓝图以Markdown格式输出，开发者可以在执行前审查和修改

---

## 3. 代码生成Agent 设计

### 3.1 目标

严格遵循蓝图，以TDD方式生成符合规范的代码。

### 3.2 处理流程

```
BLUEPRINT.md 子任务
    │
    ▼
读取蓝图条目 ── 理解该子任务的Gherkin场景和文件规划
    │
    ▼
生成测试代码 ── 根据Gherkin场景编写测试
    │
    ▼
运行测试 ── 确认测试失败（红色阶段）
    │
    ▼
生成实现代码 ── 编写最简实现使测试通过
    │
    ▼
运行测试 ── 确认测试通过（绿色阶段）
    │
    ▼
本地验证 ── 运行完整测试套件确认无回归
```

### 3.3 TDD 约束规则

代码生成Agent内置的硬约束：

1. **禁止跳过测试**: 每个子任务必须先写测试
2. **禁止修改测试**: 实现阶段不能修改测试文件
3. **最简实现**: 只写使测试通过的最少代码，不允许超出蓝图范围的"额外改进"
4. **单文件变更**: 每次只修改蓝图指定的文件
5. **必须编译通过**: 生成代码后必须确认编译/语法无误

### 3.4 关键设计点

- **蓝图绑定**: 代码生成Agent不接受任何蓝图之外的指令
- **上下文窗口管理**: 大型代码文件需要智能截取相关部分发送给LLM
- **增量生成**: 支持基于现有代码的增量修改，而非全量重写

---

## 4. 代码审查Agent 设计

### 4.1 目标

审查生成的代码，确保与蓝图一致、符合代码规范、测试充分。

### 4.2 审查维度

| 维度 | 检查内容 | 严重级别 |
|------|---------|---------|
| **蓝图一致性** | 实现是否完整覆盖蓝图中的Gherkin场景 | 阻塞 |
| **测试覆盖** | 是否每个场景都有对应测试 | 阻塞 |
| **代码风格** | 是否与项目现有风格一致 | 建议 |
| **安全性** | 是否有SQL注入、XSS等漏洞 | 阻塞 |
| **性能** | 是否有N+1查询、内存泄漏等 | 警告 |
| **边界处理** | 空值、异常情况的处理 | 建议 |

### 4.3 输出格式

```json
{
  "verdict": "REQUEST_CHANGES | APPROVE",
  "summary": "审查摘要",
  "issues": [
    {
      "severity": "blocking | warning | suggestion",
      "dimension": "blueprint-consistency",
      "file": "src/routes/auth.js",
      "line": 42,
      "description": "缺少对空密码的校验，对应的Gherkin场景未覆盖",
      "suggestion": "添加 if (!password) 判断并返回400错误"
    }
  ],
  "testResults": {
    "passed": 12,
    "failed": 0,
    "coverage": 85.5
  }
}
```

### 4.4 关键设计点

- **多次修订限制**: 最多允许3次"审查→修改"循环，超过则触发人工介入
- **只读模式**: 审查Agent绝对不能修改代码
- **上下文感知**: 审查时应理解项目的整体架构，避免给出与项目风格不符的建议

---

## 5. PR创建Agent 设计

### 5.1 目标

自动汇总所有变更，创建结构良好的 Pull Request。

### 5.2 PR 描述模板

```markdown
## 概述
[从蓝图中提取的功能描述]

## 变更内容
| 文件 | 操作 | 说明 |
|------|------|------|
| src/models/user.js | 新增 | 用户数据模型 |
| src/routes/auth.js | 修改 | 添加登录接口 |

## 验收条件
- [x] Given 有效凭据 When POST /api/auth/login Then 返回200
- [x] Given 无效凭据 When POST /api/auth/login Then 返回401

## 测试结果
- 新增测试: 8
- 全部通过: ✅
- 覆盖率: 85.5%

## 蓝图引用
[BLUEPRINT.md#登录功能](link)

## 自动生成的PR
> 此PR由 SmartPR 自动生成。如有疑问，请查看蓝图或联系开发者。
```

### 5.3 关键设计点

- **PR不自动合并**: 最终合并权始终在人类开发者手中
- **变更可追溯**: PR描述中明确标注每个变更对应的蓝图条目
- **失败处理**: 如果PR创建失败（如冲突），系统提供清晰的错误信息和解决建议

---

## 6. 通信总线设计

### 6.1 事件类型

```
TASK_ASSIGNMENT    — 分配任务给Agent
TASK_RESULT        — Agent返回任务结果
QUERY              — Agent向上游提问
RESPONSE           — 对QUERY的回复
ERROR              — 任务执行失败
HEARTBEAT          — Agent存活检测
HUMAN_ESCALATION   — 升级到人工处理
```

### 6.2 状态管理

每个任务通过 `correlationId` 追踪完整的生命周期：

```
CREATED → ASSIGNED → IN_PROGRESS → COMPLETED
                                  → FAILED
                                  → WAITING_HUMAN
```

---

*本文档随项目演进而更新。*
