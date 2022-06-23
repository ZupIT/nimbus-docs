# Components
Components are the main structure in Nimbus. Nimbus use a tree to represent the UI and the components are the node of this tree.

A component in Nimbus is the same as a component in most UI frameworks. Examples of components are: text, text-input, tab-bar, radio-button,
select-menu, data-input, row, column, button, etc.

Imagine a UI with a column containing a welcoming text at the top and a button below it. It could be represented with the following structure.

- Column
  - Text
  - Button

Each of the elements of the tree above is a component. To be able to render a UI tree declared in the backend in the frontend, Nimbus serializes the
UI tree into a JSON and deserializes it into the actual components before rendering it.

A Nimbus component is defined by the following properties:

- `_:component`: required. The name that identifies the component. It must follow the format: namespace:name. The only components that don't need a
namespace are the default components that come with any Nimbus implementation. Examples: if, forEach, layout:text, material:button, layout:column.
- `id`: a unique id for the node (considering the tree). When not specified, a random id is assigned by the frontend.
- `properties`: a map with the properties for the component. If this is a text, for instance, this could be "text: string", "fontSize: double",
"fontWeight: integer".
- `state`: declares a [state](/state) visible to this node and its descendants.
- `children`: the nodes that are children of this node. When absent, null or an empty array, this node is a leaf.

# Using components
To create a UI with components, we use the backend. See the examples below for creating the UI described in the previous section: a column with a
welcoming text and a button to go to another screen.

> Attention: the components used in the examples below are not available in the main Nimbus library.

Notice that `material:button`, in the examples below, has the property `onPress`. This is an event and it can be assigned to actions. To know more
about actions, please check [this topic](/action.md).

## Plain JSON
```json
{
  "_:component": "layout:column",
  "children": [
    {
      "_:component": "layout:text",
      "properties": {
        "text": "Welcome to our app!"
      }
    },
    {
      "_:component": "material:button",
      "properties": {
        "text": "Go to next page",
        "onPress": [{
          "_:action": "push",
          "properties": {
            "url": "/next"
          }
        }]
      }
    }
  ]
}
```

## Node backend (Typescript)

```jsx
const screen: Screen = () => (
  <Column>
    <Text>Welcome to our app!</Text>
    <Button onPress={push('/next')}>Go to next page</Button>
  </Column>
)
```

# Creating components
Nimbus comes with only a few structural components. To actually build a UI, you need to create your own components or at least use an additional lib
that provides such components.

New components must be implemented in the frontend. For backend applications, these components must only be declared, so we can use them.

Let's see an example for creating a button component.

## Frontend

### Nimbus for Compose (Android)
TODO: Borracha: Mostrar e explicar um exemplo de como criar um simples componente de botão com enabled?, text e onPress?.
Escrever em inglês

### Nimbus for SwiftUI (iOS)
TODO: Borracha: Mostrar e explicar um exemplo de como criar um simples componente de botão com enabled?, text e onPress?.
Escrever em inglês

## Backend

### JSON
If you're declaring your UI through a JSON, directly, you don't need to do anything special, just use the component you defined in the frontend.

```json
{
  "_:component": "myApp:button",
  "properties": {
    "text": "Hello World!",
    "onPress": [{
      "_:action": "log",
      "message": "Hello World!"
    }]
  }
}
```

### Nimbus Backend for Node (Typescript)
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

```typescript
import { NimbusJSX, WithChildren, FC } from '@zup-it/nimbus-backend-core'

export const Column: FC<WithChildren> = ({ id, children }) => (
  <component namespace="myApp" name="button" id={id}>
    {children}
  </component>
)
```

Now, to use the component, just import it in your screen:

```typescript
import { NimbusJSX, Screen } from '@zup-it/nimbus-backend-core'
import { log } from '@zup-it/nimbus-backend-core/actions'
import { Button } from './components/button'

const screen: Screen = () => (
  <Button text="Hello World!" onPress={log('Hello World!')} />
)
```

Notice that in the backend we just need to declare a component exists and tell what its attributes are. The actual implementation of the component is
done in the frontend application.
