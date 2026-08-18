# Feishu Bot Builder Skill

[English](README.md) | [简体中文](README.zh-CN.md)

A Codex skill for building production-ready Feishu/Lark application bots end to end. It covers developer-console configuration, interactive message cards, scheduled reporting, group `@bot` replies, secure deployment, and live verification.

## What it covers

- Choose between an enterprise self-built app bot and a custom webhook bot.
- Configure the bot capability, permissions, event subscriptions, and app versions.
- Send interactive message cards to a target chat.
- Run timezone-aware schedules with durable deduplication and catch-up behavior.
- Receive group mentions through the official SDK long connection.
- Deploy separate scheduler and mention-listener services with systemd.
- Validate delivery, connection health, restart behavior, and a real group mention.

## Minimal permissions for group mentions

For scheduled cards plus group `@bot` responses, the normal minimal permission set is:

- `im:message:send_as_bot` — send messages as the bot.
- `im:message.group_at_msg:readonly` — receive only group messages that mention the bot.
- `im:chat:readonly` — optional; needed only when discovering chats through the API.

The mention event is `im.message.receive_v1`, delivered using **Use long connection to receive events**. The skill explicitly avoids the broader permission for reading all group messages unless the user requires it.

## Install

Authenticate GitHub CLI for the private repository, then clone it into the Codex skills directory:

```bash
gh repo clone 255isWhite/feishu-build-bot-skill ~/.codex/skills/feishu-build-bot
```

Start a new Codex task after installation so the skill catalog is refreshed.

## Example requests

```text
Build a Feishu bot that sends a data-status card every day at 10:00,
14:00, and 18:00 Asia/Shanghai and replies with a fresh card when
mentioned in the target group.
```

```text
Review my Feishu app permissions and event subscriptions. Keep only the
minimum permissions needed for scheduled cards and group @bot replies.
```

## Repository layout

| Path | Purpose |
| --- | --- |
| `SKILL.md` | Main workflow, routing, safety gates, and completion criteria |
| `references/platform-setup.md` | Developer-console setup, publishing, and exact mention permission flow |
| `references/implementation.md` | Card delivery, scheduling, long-connection listener, and deduplication |
| `references/deployment-checklist.md` | systemd deployment, hardening, validation, and troubleshooting |
| `agents/openai.yaml` | Skill display metadata |

## Security model

The workflow keeps App Secrets, tokens, chat IDs, connection tickets, SSH passwords, and tenant-specific identifiers out of source control and chat logs. Production secrets belong in a protected server environment file with mode `0600`. Permission enablement and application publishing are treated as explicit security gates that require review before the final action.
