# log
Logs a message. How the message is logged id defined by the [Logger](TODO_link).

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
