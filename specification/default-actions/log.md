# log
Logs a message to the platform console. If the frontend library allows for log customization, this Logger will be used.

The properties of the log action must follow the structure:

```typescript
interface SetStateProperties {
  message: string,
  level?: 'Info' | 'Warning' | 'Error',
}
```

Where:
- `message`: the text to log.
- `level`: optional. The severity of the message. Default is `'Info'`.

## Example:
```json
{
  "_:action": "log",
  "properties": {
    "message": "This is a warning message!",
    "level": "Warning"
  }
}
```

The example above logs a warning message.
