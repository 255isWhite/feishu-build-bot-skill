# Production deployment checklist

## Files and secrets

- Install application code under a stable path such as `/opt/<service>`.
- Create a dedicated virtual environment and install a bounded `lark-oapi` version.
- Store environment configuration under `/etc/<service>/env`, owned by root with mode `0600`.
- Keep the App Secret, webhook token, and signing secret out of source control.
- Configure the source URL, dashboard URL, timezone, report slots, target chat ID, timeouts, and mention enablement through environment variables.

## systemd

Create separate units for the scheduler and mention listener. Both should:

- Start after and want `network-online.target`.
- Use `Restart=always` with a short restart delay.
- Be enabled for `multi-user.target`.
- Use a low-privilege or dynamic user.
- Set `NoNewPrivileges=yes`, `PrivateTmp=yes`, `ProtectSystem=strict`, and `ProtectHome=yes` where compatible.
- Allow only `AF_UNIX`, `AF_INET`, and `AF_INET6` address families.

Give only the scheduler access to the writable state directory. The mention listener should not modify schedule state.

## Safe validation sequence

1. Run unit tests and configuration parsing locally.
2. On the host, verify data-source reachability without sending a message.
3. Preview the generated card.
4. Verify the installed SDK imports successfully.
5. Enable and start both services.
6. Confirm `active` and `enabled` for both units.
7. Confirm scheduler logs show the intended timezone and every report slot.
8. Confirm the long connection is established without printing the connection URL.
9. Send one approved live card and require Feishu `code: 0`.
10. After the app version is published, ask a user to `@bot` in the target group and confirm a fresh card is returned.

## Troubleshooting

- **Long-connection verification fails:** start the SDK client first; check outbound network access and app credentials.
- **Event exists but no callback arrives:** confirm `im.message.receive_v1`, the published app version, and `im:message.group_at_msg:readonly`.
- **Callback arrives but no card is sent:** confirm chat allow-list filtering, outbound permission, and the worker queue.
- **Duplicate cards:** deduplicate scheduled sends by slot and mention events by message ID.
- **App configuration appears saved but ineffective:** create and publish a new version; unpublished console changes do not affect the live app.
- **Sensitive connection data appears in logs:** lower SDK logging to `ERROR` and avoid reproducing historical credential-bearing lines.
