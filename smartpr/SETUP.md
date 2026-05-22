# SmartPR 搭建指南

**适用于**: 希望从源码运行或参与开发的贡献者

---

## 环境要求

| 依赖 | 版本要求 | 说明 |
|------|---------|------|
| Node.js | ≥ 20.0.0 | 运行时环境 |
| Git | ≥ 2.40 | 版本控制 |
| GitHub CLI (`gh`) | 最新版 | 用于创建PR和管理仓库 |

### LLM API Key (至少一个)

根据你选择使用的模型后端，准备对应的API Key：

| 提供商 | 环境变量 | 获取地址 |
|--------|---------|---------|
| Anthropic Claude | `ANTHROPIC_API_KEY` | https://console.anthropic.com |
| Google Gemini | `GEMINI_API_KEY` | https://aistudio.google.com/apikey |
| OpenAI | `OPENAI_API_KEY` | https://platform.openai.com/api-keys |
| DeepSeek | `DEEPSEEK_API_KEY` | https://platform.deepseek.com |

### 项目管理集成 (可选)

| 工具 | 环境变量 | 说明 |
|------|---------|------|
| Jira | `JIRA_API_TOKEN`, `JIRA_EMAIL`, `JIRA_DOMAIN` | 从Jira读取需求 |
| Linear | `LINEAR_API_KEY` | 同步任务状态 |
| GitHub Issues | (使用 gh CLI 认证) | 无需额外配置 |

---

## 本地开发环境搭建

### 1. 克隆仓库

```bash
git clone https://github.com/YOUR_USERNAME/smartpr.git
cd smartpr
```

### 2. 安装依赖

```bash
npm install
```

### 3. 配置环境变量

```bash
cp .env.example .env
# 编辑 .env 文件，填入你的 API Key
```

`.env` 文件示例：

```env
# LLM API Keys (至少配置一个)
ANTHROPIC_API_KEY=sk-ant-xxxxxxxxxxxxx
GEMINI_API_KEY=xxxxxxxxxxxxx
OPENAI_API_KEY=sk-xxxxxxxxxxxxx

# 项目管理 (可选)
LINEAR_API_KEY=lin_api_xxxxxxxxxxxxx

# 首选模型配置
SMARTPR_PLANNER_MODEL=claude-opus-4-7
SMARTPR_CODER_MODEL=claude-sonnet-4-6
SMARTPR_REVIEWER_MODEL=gemini-3-flash-preview
```

### 4. 初始化项目配置

```bash
# 在你的目标仓库中运行
node src/cli.js init
node src/cli.js config
```

### 5. 验证安装

```bash
node src/cli.js doctor
```

预期输出:

```
✅ Node.js 20.15.0
✅ Git 2.45.0
✅ GitHub CLI 2.50.0
✅ ANTHROPIC_API_KEY configured
✅ smartpr.config.js valid
✅ .github/workflows installed
✅ Templates up to date

SmartPR is ready!
```

---

## 使用GitHub Actions (CI/CD)

SmartPR 通过 GitHub Actions 实现自动化流水线。初始化后，以下工作流文件会自动安装到 `.github/workflows/` 目录：

### orchestrator.yml
监听 Story 文件变更，自动触发从需求到代码的完整流水线。

```yaml
# 触发条件: push 到包含 STORY.md 的分支
# 执行: 需求理解 → 蓝图规划 → 代码生成 → 审查 → PR创建
```

### reviewer.yml
当 Agent 创建的 PR 被提交时，自动触发代码审查。

```yaml
# 触发条件: 标记为 smartpr-generated 的 PR
# 执行: 审查代码与蓝图的一致性
```

### nightly.yml
每日定时运行完整测试套件，并在失败时创建修复任务。

```yaml
# 触发条件: 每日 03:00 UTC
# 执行: 运行测试套件 → 失败时创建Issue
```

### 配置 GitHub Secrets

在仓库的 Settings → Secrets and variables → Actions 中添加：

| Secret 名称 | 说明 |
|------------|------|
| `ANTHROPIC_API_KEY` | Claude API 密钥 |
| `GEMINI_API_KEY` | Gemini API 密钥 |
| `LINEAR_API_KEY` | Linear API 密钥 (可选) |
| `GH_TOKEN` | GitHub Token (通常自动提供) |

---

## 项目配置参考

完整的 `smartpr.config.js` 配置项：

```javascript
module.exports = {
  // 仓库信息
  repo: 'your-org/your-repo',

  // 模型选择
  plannerModel: 'claude-opus-4-7',      // 蓝图规划 (需要强推理能力)
  coderModel: 'claude-sonnet-4-6',      // 代码生成 (需要速度)
  reviewerModel: 'gemini-3-flash-preview', // 代码审查 (性价比)
  understanderModel: 'claude-haiku-4-5', // 需求理解 (轻量任务)

  // 自动化策略
  autoMergeBlueprints: false,           // 蓝图PR是否需要人工审批
  maxReviewerRevisions: 3,              // 最多修复-审查循环次数
  requireHumanApproval: true,           // 最终PR是否需要人工审批

  // 测试策略
  testFramework: 'auto',                // auto | jest | vitest | pytest | go-test
  minCoverageThreshold: 80,             // 最低测试覆盖率 (%)

  // 项目管理
  pmTool: 'github-issues',             // jira | linear | github-issues
  pmConfig: {
    // 根据所选工具配置
  },

  // 通知
  notifyOnPR: true,
  notifyOnFailure: true,
};
```

---

*如有搭建问题，请提交 [Issue](https://github.com/YOUR_USERNAME/smartpr/issues)。*
