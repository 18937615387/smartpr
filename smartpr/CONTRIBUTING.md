# 贡献指南

感谢你对 SmartPR 的关注！本文档帮助你了解如何参与贡献。

## 行为准则

本项目遵循 [Contributor Covenant 行为准则](CODE_OF_CONDUCT.md)。

## 如何贡献

### 报告Bug

1. 在 [Issues](https://github.com/YOUR_USERNAME/smartpr/issues) 中搜索是否已有相同报告
2. 使用 Bug Report 模板提交新Issue
3. 附上 Node.js 版本、操作系统信息和 `smartpr doctor` 的输出

### 提出功能建议

1. 提交 Feature Request Issue
2. 描述功能解决什么问题，而非仅描述功能本身
3. 说明该功能如何契合 SmartPR 的 需求→蓝图→代码→审查→PR 流水线

### 提交代码

1. Fork 仓库并从 `main` 分支创建新分支
2. 编码时遵循项目风格 (JavaScript, JSDoc, 无外部依赖)
3. 确保新增功能包含文档说明
4. 填写 Pull Request 模板
5. 提交 PR 并等待审核

## 开发环境搭建

```bash
git clone https://github.com/YOUR_USERNAME/smartpr.git
cd smartpr
npm install
cp .env.example .env
# 编辑 .env 填入 API Key
```

## 代码风格

- **仅使用 JavaScript**，不使用 TypeScript
- **JSDoc 注释**所有导出函数
- **配置驱动**，所有仓库特定值从 `smartpr.config.js` 读取
- **零外部依赖**，优先使用 Node.js 内置模块

## 许可证

贡献的代码将基于 Apache License 2.0 授权。
