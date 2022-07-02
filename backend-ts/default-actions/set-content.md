# setContent
Check the [specification](/specification/default-actions/set-content.md) for the full definition of this action.

## Properties
```typescript
export interface SetContentParams {
  /**
   * The id of the component to receive the new elements.
   */
  id: string,
  /**
   * The component (node) to add.
   */
  value: Expression<Component>,
  /**
   * The mode to attach the new nodes
   * @defaultValue `'Append'`
   */
  mode?: 'Append' | 'Prepend' | 'Replace' | 'ReplaceItself',
}
```

## Example
In the code below, we add a new row to the UI whenever a button is clicked:

```tsx
import { NimbusJSX } from '@zup-it/nimbus-backend-core'
import { setContent } from '@zup-it/nimbus-backend-core/actions'
import { Screen } from '@zup-it/nimbus-backend-express'
import { Column } from '@zup-it/nimbus-backend-layout'
import { Button } from '../components/button' // this component's implementation is not important for this example
import { TextInput } from '../components/text-input' // this component's implementation is not important for this example

const MyScreen: Screen = () => (
  <Column>
    <Column id="inputGroup">
      <TextInput placeholder="name" />
    </Column>
    <Button onPress={setContent({ id: 'inputGroup', value: <TextInput placeholder="name" /> })}>
      Add more
    </Button>
  </Column>
)
```