# YCloud Developer Kit

适用版本：0.7.8

用自然语言规划 YCloud 集成、生成服务端代码，并通过本地模拟测试检查流程。整套包含 17 个 Skill，覆盖 88 个 API 操作及鉴权、Webhook 接收两项跨领域能力。

本仓库发布可公开的技能、参考资料和使用说明，使用独立的发行历史。

## 一次安装整套技能

在目标项目目录运行，选择要使用的 AI 工具：

```bash
npx skills add YCloud-Developers/YCloud-Developer-Kit --skill '*'
```

WorkBuddy 项目使用 CodeBuddy 兼容的技能目录：

```bash
npx skills add YCloud-Developers/YCloud-Developer-Kit --skill '*' -a codebuddy --copy -y
```

Claude Code：

```bash
npx skills add YCloud-Developers/YCloud-Developer-Kit --skill '*' -a claude-code --copy -y
```

Codex：

```bash
npx skills add YCloud-Developers/YCloud-Developer-Kit --skill '*' -a codex --copy -y
```

这些命令在当前项目中安装全部 17 个 Skill。WorkBuddy 会读取项目下的 `.codebuddy/skills/`。在新会话中使用 `ycloud-developer-kit-smoke-test` 检查安装，跨领域需求使用 `ycloud-integration-architect`。

## 插件总包

[下载 0.7.8](https://github.com/YCloud-Developers/YCloud-Developer-Kit/releases/tag/v0.7.8)：Codex、Claude Code 和 WorkBuddy 各提供一个完整插件 ZIP，每个都包含全部 17 个 Skill。发行附件同时提供中文产品指南、分发清单及 SHA256SUMS。

## 使用边界

Skill 用于集成设计、代码生成和本地模拟验证。Skill 执行中不读取真实凭据或业务数据，不调用实时 YCloud API。生成的应用使用项目自己的服务端运行时配置。模拟测试成功不代表真实消息送达或生产部署完成。

不包含 SMS、Verify、Voice 和 Email 集成。安装文件完整性已验证；具体宿主中的加载和对话行为仍需按实际环境验收。

[官网](https://www.ycloud.com/) · [API 文档](https://docs.ycloud.com/reference) · [客户支持](https://www.ycloud.com/customer-support)

License: Apache-2.0
