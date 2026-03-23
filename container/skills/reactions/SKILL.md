# Reactions

You can react to messages with emoji using the `react_to_message` MCP tool.

## When to use

- Acknowledge a message quickly (thumbs up, checkmark)
- Signal progress or status (eyes, hourglass, checkmark)
- Express sentiment without a full reply

## How to react

### React to the most recent message

```
react_to_message(emoji: "👍")
```

### React to a specific message

```
react_to_message(emoji: "✅", message_id: "12345")
```

To find a message ID, query the database:
```bash
sqlite3 /workspace/store/messages.db "SELECT id, content FROM messages WHERE chat_jid = '$NANOCLAW_CHAT_JID' ORDER BY timestamp DESC LIMIT 5;"
```

### Remove a reaction

```
react_to_message(emoji: "")
```

## Common emoji

| Emoji | Meaning |
|-------|---------|
| 👍 | Acknowledged / agreed |
| 👀 | Looking into it |
| ✅ | Done / completed |
| ❌ | Failed / rejected |
| 🔄 | Working on it |
| 💭 | Thinking |
| ❤️ | Liked |
