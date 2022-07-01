# condition
Check the [specification](/specification/default-actions/condition.md) for the full definition of this action.

## Properties
```typescript
interface ConditionalActionParams {
  /**
   * The condition to verify.
   */
  condition: DynamicExpression<boolean>,
  /**
   * The actions to run if the condition turns out to be true.
   */
  onTrue?: Actions,
  /**
   * The actions to run if the condition turns out to be false.
   */
  onFalse?: Actions,
}
```

## Example
```tsx
import { NimbusJSX } from '@zup-it/nimbus-backend-core'
import { conditionalAction, log } from '@zup-it/nimbus-backend-core/actions'
import { gte } from '@zup-it/nimbus-backend-core/operations'
import { Button } from '../components/button' // this component's implementation is not important for this example

// ...
// suppose we have the state user with the property age (number)

const MyScreen: Screen = () => (
  { /* ... */ }
  <Button onPress={conditionalAction(gte(user.get('age'), 18), log("You're an adult"), log("You're underage"))}>
    Click me!
  </Button>
)
```

The button in the screen above, when pressed, logs "You're an adult" if the value of `user.age` is 18 or more. Otherwise, it logs "You're underage".
