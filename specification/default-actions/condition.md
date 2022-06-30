# condition
This action is very simple, it verifies a condition and if it's true, it triggers the event "onTrue", if it's false, it triggers the event "onFalse".
This is useful for triggering actions based on a condition.

The properties for the action `condition` are:
```typescript
interface ConditionProperties {
  condition: boolean,
  onTrue?: Action[],
  onFalse?: Action[],
}
```

Where:
- `condition` is the condition you want to check.
- `onTrue` is the event triggered if `condition` is true.
- `onFalse` is the event triggered if `condition` is false.

## Example
Suppose a state called "user" with the property "age".

```json
{
  "_:action": "condition",
  "properties": {
    "condition": "@{gte(user.age, 18)}",
    "onTrue": [
      {
        "_:action": "log",
        "message": "You're an adult"
      }
    ],
    "onTrue": [
      {
        "_:action": "log",
        "message": "You're underage"
      }
    ]
  }
}
```

The action above logs "You're an adult" if the value of `user.age` is 18 or more. Otherwise, it logs "You're underage".
