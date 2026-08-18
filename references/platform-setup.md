# Feishu developer-console setup

## Application bot

1. Open the Feishu developer console and create or select an enterprise self-built app.
2. Add the **Bot** capability.
3. Record the App ID. Reveal the App Secret only when it can be written directly to the protected server environment; never return it in chat or logs.
4. Enable the narrow outbound permission `im:message:send_as_bot`.
5. Enable `im:chat:readonly` only if chat discovery through `GET /im/v1/chats` is required.
6. Create and publish a baseline version.
7. Add the bot to the target group, list its chats, and store the target `chat_id` as deployment configuration.

## Group mention events

Use the following exact configuration:

- Subscription transport: **Use long connection to receive events**.
- Event: **Receive message**, `im.message.receive_v1`.
- Minimal application-identity permission: **Get messages where users @ the bot in group chats**, `im:message.group_at_msg:readonly`.

Order matters:

1. Install the official SDK and start the long-connection client with the app credentials.
2. Confirm the service is actively connected.
3. In **Events & Callbacks**, choose long connection and click **Verify**.
4. Save only after the console shows that the connection succeeded.
5. Add `im.message.receive_v1`.
6. If the console recommends the broader “get messages sent by users and bots in groups” permission, choose **do not enable now**.
7. Open **Permission Management**, search for `im:message.group_at_msg:readonly`, select exactly that permission, and stop before the final **Confirm enable permission** action.
8. Obtain user confirmation naming the permission and its effect. Then enable it.
9. Create a new version, review the diff, and publish it so the event and permission become effective.

## Browser-operation rules

- Use the browser-control skill and the user's authenticated session when available.
- Inspect the DOM after each navigation, selection, save, or publish; rely on the visible success state rather than assuming a click worked.
- Save the subscription transport before adding events.
- Adding an event may open a permission recommendation dialog. Decline the broad recommendation and select the minimal permission manually.
- Preserve the live tab as a handoff when stopped at a required user confirmation.

## Expected published permissions

Typical minimal set for scheduled cards plus group mentions:

- `im:message:send_as_bot`
- `im:message.group_at_msg:readonly`
- `im:chat:readonly` only when chat listing remains necessary

Do not add full group-message reading unless the user explicitly requires processing non-mention messages.
