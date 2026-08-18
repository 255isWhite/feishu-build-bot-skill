---
name: feishu-build-bot
description: Create, configure, publish, deploy, and troubleshoot Feishu/Lark bots that send interactive message cards, run on schedules, and respond when mentioned in group chats. Use for Feishu developer-console setup, enterprise self-built app bots, custom-bot versus app-bot selection, message-card delivery, lark-oapi long connections, im.message.receive_v1 subscriptions, minimal @-message permissions, chat discovery, systemd deployment, or end-to-end bot verification.
---

# Build a Feishu Bot

Build the bot end to end: choose the correct bot type, configure the Feishu app, implement card delivery and mention events, deploy a durable service, publish the app, and verify the live behavior.

## Route the work

- Read [platform-setup.md](references/platform-setup.md) before changing the Feishu developer console, permissions, events, or application versions.
- Read [implementation.md](references/implementation.md) when creating or reviewing the bot service.
- Read [deployment-checklist.md](references/deployment-checklist.md) when installing on a server, configuring systemd, or validating production.

Read every reference that matches the task. Keep credentials and tenant-specific identifiers out of generated source and documentation.

## Choose the bot type

Use an enterprise self-built app bot when any of these are required:

- Receive group `@bot` messages.
- Send to a chat by `chat_id`.
- Subscribe to events through an SDK long connection.
- Manage permissions and publish application versions.

Use a custom webhook bot only for outbound-only cards where event reception is unnecessary. Do not promise group mention handling with a webhook-only bot.

## Execute the workflow

1. Confirm the target tenant, group, data source, reporting schedule and timezone, deployment host, and whether group mentions must work.
2. Inspect existing code, app configuration, and deployment before changing anything. Preserve unrelated state.
3. Configure or create the app bot following `platform-setup.md`.
4. Implement outbound cards and scheduled-slot deduplication following `implementation.md`.
5. If mentions are needed, establish the SDK long connection before saving the long-connection subscription mode. Then add the receive-message event and request only the minimal mention permission.
6. Deploy and harden the services following `deployment-checklist.md`.
7. Publish a new app version after reviewing the exact pending permissions and event changes.
8. Verify outbound delivery, scheduled state, long-connection health, and one real `@bot` interaction in the target group.

## Safety gates

- Never print, paste into chat, commit, or log an App Secret, webhook token, signing secret, SSH password, access token, connection ticket, or temporary access key.
- Store production secrets in a root-owned environment file with mode `0600`; output only key names when inspecting it.
- Treat developer-console permission changes as security-sensitive. Stop immediately before the final permission-enabling click and obtain action-time user confirmation that names the exact permission.
- Prefer `im:message.group_at_msg:readonly` for `@bot` behavior. Do not accept the broader group-message permission merely because the console recommends it.
- Do not publish until the pending version diff contains only the intended capabilities, events, and permissions.
- Avoid sending test messages unless the user requested or approved a live test.
- Restrict mention handling to configured target chat IDs unless the user explicitly wants broader scope.

## Completion criteria

Report the bot complete only when:

- The intended app version is published and enabled.
- The bot is present in the target group.
- Outbound card delivery returns Feishu success (`code: 0`).
- Every scheduled time has an independent deduplication slot.
- The mention listener is enabled, active, connected, subscribed to `im.message.receive_v1`, and authorized with the minimal permission.
- A real group mention produces a fresh status card.
- Services are configured for restart and boot persistence, and no secret appears in source or logs.
