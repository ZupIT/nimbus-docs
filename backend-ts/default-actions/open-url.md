# openUrl
Check the [specification](/specification/default-actions/open-url.md) for the full definition of this action.

## Properties
```typescript
interface OpenUrlParams {
  /**
   * The URL to open.
   */
  url: Expression<string>,
}
```

## Example
```tsx
import { NimbusJSX } from '@zup-it/nimbus-backend-core'
import { openUrl } from '@zup-it/nimbus-backend-core/actions'
import { Screen } from '@zup-it/nimbus-backend-express'
import { Button } from '../components/button' // this component's implementation is not important for this example

const MyScreen: Screen = () => (
  <Button onPress={openUrl('https://www.google.com.br')}>
    Go to Google
  </Button>
)
```

The button in the screen above, when pressed, opens Google in the default web browser.
