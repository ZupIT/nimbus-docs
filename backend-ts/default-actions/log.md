# log
Check the [specification](/specification/default-actions/log.md) for the full definition of this action.

## Properties
```typescript
interface LogParams {
  /**
   * The message to log.
   */
  message: Expression<string>,
  /**
   * The severity of the log message. Defaults to 'Info'.
   */
  level?: LogLevel,
}
```

## Example
```tsx
import { NimbusJSX } from '@zup-it/nimbus-backend-core'
import { log } from '@zup-it/nimbus-backend-core/actions'
import { Screen } from '@zup-it/nimbus-backend-express'
import { Button } from '../components/button' // this component's implementation is not important for this example

const MyScreen: Screen = () => (
  <Button onPress={log({ message: 'This is a warning message!', level: 'Warning' })}>
    Log a waning
  </Button>
)
```

The button in the screen above, when pressed, logs the warning: "This is a warning message!".
