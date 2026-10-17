# YCloud Developer Kit

适用版本：0.7.13

用自然语言规划 YCloud 集成、生成服务端代码、检查本地模拟流程，并按请求执行真实 YCloud 操作。整套包含 17 个 Skill，覆盖 88 个 API 操作及鉴权、Webhook 接收两项跨领域能力。

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

[下载 0.7.13](https://github.com/YCloud-Developers/YCloud-Developer-Kit/releases/tag/v0.7.13)：Codex、Claude Code 和 WorkBuddy 各提供一个完整插件 ZIP，每个都包含全部 17 个 Skill。发行附件同时提供中文产品指南、分发清单及 SHA256SUMS。

## 使用边界

Skill 支持集成设计、代码生成、本地模拟验证，以及用户请求和已有授权范围内的真实 API 操作。沿用已明确的账户、环境、范围和服务端凭据引用，不因真实调用或领域交接重复索要授权；必要目标不清楚、操作超出范围，或破坏性影响尚未授权时再补充询问。

只读取任务需要的配置和业务数据，不在对话、源码、日志或前端中暴露密钥。模拟和 Sandbox 任务保持合成数据与指定端点，不自动切换到真实服务；本地开发请求不授权无关真实操作。真实变更超时后先检查可用状态，不盲目重试。安装 smoke 检查仍无外部副作用，模拟测试成功不代表真实消息送达或生产部署完成。

不包含 SMS、Verify、Voice 和 Email 集成。安装文件完整性已验证；具体宿主中的加载和对话行为仍需按实际环境验收。

[官网](https://www.ycloud.com/) · [API 文档](https://docs.ycloud.com/reference) · [客户支持](https://www.ycloud.com/customer-support)

License: Apache-2.0
