# setState
Check the [specification](/specification/default-actions/set-state.md) for the full definition of this action.

In Nimbus Backend TS, we don't use this action directly. Instead, we use the method `set` of a state object. See the example below:

## Examples
```tsx
import { NimbusJSX, createState } from '@zup-it/nimbus-backend-core'
import { Screen } from '@zup-it/nimbus-backend-express'
import { Column, Text } from '@zup-it/nimbus-backend-layout'
import { Button } from '../components/button' // this component's implementation is not important for this example

interface User {
  name: string,
  age: number,
}

const user = createState('user', { name: 'John', age: 40 })

const MyScreen: Screen = () => (
  <Column>
    <Text>{user.get('name')} is {user.get('age')} years old.</Text>
    <Button onPress={user.get('name').set('Joseph')}>
      Set the name to "Joseph"
    </Button>
    <Button onPress={user.get('age').set(32)}>
      Set the age to 32
    </Button>
    <Button onPress={user.set({ name: 'Mary', age: 16 })}>
      Set the name to "Mary" and the age to 16
    </Button>
  </Column>
)
```
