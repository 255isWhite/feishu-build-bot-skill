# Implementation reference

## Outbound card flow

For an app bot:

1. Request a tenant token with `POST /open-apis/auth/v3/tenant_access_token/internal` using App ID and App Secret.
2. Send a card with `POST /open-apis/im/v1/messages?receive_id_type=chat_id`.
3. Set `receive_id` to the target chat ID, `msg_type` to `interactive`, and `content` to the JSON-encoded card object.
4. Treat only Feishu `code: 0` as success. Persist scheduling state only after success.

Use card schema `2.0`. Keep status, timestamps, failures, and a dashboard link visible. Escape untrusted source text before placing it in markdown.

## Scheduling

Represent reporting times as sorted `(hour, minute)` slots in one timezone, for example `10:00,14:00,18:00` in `Asia/Shanghai`.

Persist `last_sent_slot` as `YYYY-MM-DD@HH:MM`. At each poll:

1. Build today's configured datetimes.
2. Select the latest due slot.
3. Send only when that slot is newer than `last_sent_slot`.
4. On catch-up after downtime, send the latest missed slot once instead of replaying every older slot.
5. Do not consume a scheduled slot for manual sends or mention-triggered sends.

Use atomic state-file replacement. Keep `last_snapshot` for “since previous report” deltas.

## Mention listener

Use the official Python SDK:

```python
import lark_oapi as lark

handler = (
    lark.EventDispatcherHandler.builder("", "")
    .register_p2_im_message_receive_v1(callback)
    .build()
)
client = lark.ws.Client(
    app_id,
    app_secret,
    event_handler=handler,
    log_level=lark.LogLevel.ERROR,
)
client.start()
```

The callback must return quickly:

1. Extract `event.message.chat_id`, `message_id`, and `chat_type`.
2. Accept only `chat_type == "group"` and an allow-listed target `chat_id`.
3. Deduplicate by `message_id`.
4. Put a small job on a bounded queue and return.
5. In a background worker, collect fresh data and send a card to the event's chat ID.
6. Do not write scheduled-state markers for mention responses.

Set the SDK log level to `ERROR`; info-level connection logs may include temporary connection URLs, tickets, or access keys.

## Service structure

Prefer separate processes:

- Scheduler/collector service: polls state and sends scheduled cards.
- Mention-listener service: maintains the WebSocket connection and enqueues mention jobs.

Keep SDK imports lazy if outbound-only commands should work without the SDK installed. Unit-test slot deduplication, card rendering, destination override, chat filtering, and message-event deduplication.
