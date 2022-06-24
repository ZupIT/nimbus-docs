> Article level: basic.

# Components
Check the [specification](/specification/component.md) to know more about the definition of Nimbus Components. 

# Using components
To create a UI with components, we use the backend. See the examples below for creating the UI described in the previous section: a column with a
welcoming text and a button to go to another screen.

> Attention: the components used in the examples below are not available in the main Nimbus library.

Notice that `material:button`, in the example below, has the property `onPress`. This is an event and it can be assigned to actions. To know more
about actions, please check [this topic](/action.md).

```tsx
import { NimbusJSX } from '@zup-it/nimbus-backend-core'
import { push } from '@zup-it/nimbus-backend-core/actions'
import { Screen } from '@zup-it/nimbus-backend-express'

const MyScreen: Screen = () => (
  <Column>
    <Text>Welcome to our app!</Text>
    <Button onPress={push('/next')}>Go to next page</Button>
  </Column>
)
```

# Creating components
When using the backend lib for Typescript, you need to declare the component with its attributes:

```typescript
import { NimbusJSX, Actions, Expression, FC } from '@zup-it/nimbus-backend-core'

interface Props {
  onPress?: Actions,
  enabled?: Expression<boolean>,
  text: string,
}

export const Button: FC<ButtonProps> = ({ id, text, onPress }) => (
  <component namespace="myApp" name="button" id={id} properties={{ text, onPress }} />
)
```

- `Props` is an interface that declares which attributes can be given to a Button.
- `Actions` is a type imported from the main lib. It means this attribute is an event, i.e. it must receive an Action or a list of Actions.
- `Expression<T>` is a type imported from the main lib. It means that this attribute accepts [expressions](/expression) that results in the type `T`
or `T`. This type has a high chance of being removed before a stable release, in this case, every attribute would accept expressions by default.
- `FC<T>` means "Functional Component". This term has been borrowed from [React](https://reactjs.org/) and it represents a function that declares a
Component where the attributes are of type `T`. An `FC` receives the component properties plus an id and must return a `<component />`.
- `<component />` is the only intrinsic element of Nimbus JSX, it is the basic structure for declaring components. It must receive at least the
namespace and name of the component. It can receive an id, properties and state. Just like any other JSX, if the component accepts children, the
children must be placed inside the opening and closing tags of the component. See the example below:

```tsx
import { NimbusJSX, WithChildren, FC } from '@zup-it/nimbus-backend-core'

export const Column: FC<WithChildren> = ({ id, children }) => (
  <component namespace="myApp" name="button" id={id}>
    {children}
  </component>
)
```

Now, to use the component, just import it in your screen:

```tsx
import { NimbusJSX } from '@zup-it/nimbus-backend-core'
import { Screen } from '@zup-it/nimbus-backend-express'
import { log } from '@zup-it/nimbus-backend-core/actions'
import { Button } from './components/button'

const MyScreen: Screen = () => (
  <Button text="Hello World!" onPress={log('Hello World!')} />
)
```

Notice that in the backend we just need to declare a component exists and tell what its attributes are. The actual implementation of the component is
done in the frontend application.

# Read next
:point_right: [Action](/action)
