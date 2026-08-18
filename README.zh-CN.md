# 飞书机器人搭建 Skill

[English](README.md) | [简体中文](README.zh-CN.md)

一个用于端到端搭建生产级飞书/Lark 应用机器人的 Codex Skill。覆盖飞书开发者平台配置、消息卡片、定时播报、群聊 `@机器人` 回复、安全部署和真实环境验证。

## 功能范围

- 判断应使用企业自建应用机器人还是自定义 Webhook 机器人。
- 配置机器人能力、权限、事件订阅和应用版本。
- 向目标群聊发送交互式消息卡片。
- 按指定时区定时发送，并持久化去重和处理停机后的补发。
- 通过官方 SDK 长连接接收群聊 `@机器人` 消息。
- 使用 systemd 分别部署定时任务服务和消息监听服务。
- 验证消息发送、长连接、开机自启、故障重启和真实群聊 `@机器人` 效果。

## 群聊 @机器人的最小权限

实现定时卡片和群聊 `@机器人` 回复时，通常只需要：

- `im:message:send_as_bot`：以机器人身份发送消息。
- `im:message.group_at_msg:readonly`：仅接收群聊中提及机器人的消息。
- `im:chat:readonly`：可选；只有通过 API 查询群聊时才需要。

消息事件为 `im.message.receive_v1`，订阅方式选择 **使用长连接接收事件**。除非用户明确要求处理未提及机器人的普通群消息，否则本 Skill 不开通读取全部群消息的宽泛权限。

## 安装

先使用 GitHub CLI 登录有权访问该私有仓库的账户，再克隆到 Codex Skills 目录：

```bash
gh repo clone 255isWhite/feishu-build-bot-skill ~/.codex/skills/feishu-build-bot
```

安装后新建一个 Codex 任务，使 Skill 列表重新加载。

## 使用示例

```text
搭建一个飞书机器人，每天在 Asia/Shanghai 时区的 10:00、14:00、
18:00 发送数据状态卡片；在目标群中 @机器人时，立即采集最新状态并回复卡片。
```

```text
检查我的飞书应用权限和事件订阅，只保留定时发送卡片与群聊
@机器人回复所需的最小权限。
```

## 仓库结构

| 路径 | 用途 |
| --- | --- |
| `SKILL.md` | 主流程、任务路由、安全门禁和完成标准 |
| `references/platform-setup.md` | 开发者平台配置、发布流程和明确的 @权限开通步骤 |
| `references/implementation.md` | 卡片发送、定时调度、长连接监听和去重逻辑 |
| `references/deployment-checklist.md` | systemd 部署、服务加固、验证和故障排查 |
| `agents/openai.yaml` | Skill 展示信息 |

## 安全设计

本流程禁止将 App Secret、Token、群 ID、连接票据、SSH 密码和租户专属标识写入源码、Git 仓库或聊天日志。生产密钥应保存在服务器上权限为 `0600` 的环境文件中。开通权限和发布应用版本均作为安全门禁，执行最终操作前必须核对变更内容。
