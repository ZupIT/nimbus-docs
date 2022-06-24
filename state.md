> Article level: intermediary.

# State
Nimbus doesn't have an embedded script language, and we defend that it shouldn't have one. But we still need to make it
possible to create dynamic UIs from the backend. To solve this issue, Nimbus introduces three concepts: State,
[Operations](/operation.md) and [Actions](/action.md). This topic talks about States.

States are the Nimbus way of creating and manipulating a value that can shared across multiple components. We can make an
equivalence to variables in programming languages, they can be declared (local states), some are global (global
state) and others are injected into your code (implicit states).

**Important**: States are variables of Nimbus, they're not variables of your backend application, i.e. Nimbus States are resolved
when the code is run by the frontend, and not when the request is made to the server.

## Local states
Local states are declared by the backend application in the body of a Component. As seen in the [topic for components](/component.md), every
component can receive the attribute `state`.

A Local state is declared with 2 properties:
- `id`: required. An id to identify the state. Think this as the variable name. This is a string and it must match the regex `/^([a-z]|[A-Z]|_)\w*$/`,
i.e. it must be comprised of only letters, numbers and underscore; it can't start with a number.
- `value`: the initial value for this state, can be of any type. If not provided, null is used.

This type of state is said to be local because its scope is only the node where it's declared and its descendants. Any node above or at the same
level (siblings) won't be able to see the state.

The value of a state is accessed through an [expression](/expression.md), which is an special type of string that gets parsed by Nimbus in the
frontend before the components are rendered.

An expression string always start with `@{` and ends with `}`. To make a reference to state, you just need to write its id! See the example below:

```json
{
  "_:component": "layout:text",
  "state": {
    "id": "name",
    "value": "John",
  },
  "properties": {
    "text": "@{name}"
  }
}
```

The UI above renders a text component written "John".

> Attention: although expressions are strings, when rendered by the frontend they will be replaced by the state value. So if the state is a boolean,
the value received by the component in frontend will be a boolean.

To know more about expressions, please read the [dedicated topic](/expression.md).

### Local states in the backend for Node (Typescript)

#### Creating a local state
In a Node backend using our lib, to create a local state, you must use the function `createState`, which accepts 2 arguments:

1. The first is a string with the state id.
2. The initial value the frontend application should give to the context. This is optional and when not provided, the
state will be initialized with null.

`createState` is a generic function, i.e., it can receive a type `T` and act differently (type-wise) according to it.
Let's say we want to create a state that holds a string:

```typescript
import { createContext } from '@zup-it/nimbus-backend-core'

const username = createState<string>('username')
```

Simple right? `username` is now a State that refers to a string value.

```typescript
import { createContext } from '@zup-it/beagle-backend-core'

const username = createContext('username', 'John')
```

As long as a default value is provided, the generic type can be omitted, since it can be easily inferred by Typescript.

Let's see a more complex example of a state:

```typescript
import { NimbusJSX, createContext } from '@zup-it/nimbus-backend-core'

interface Document {
  name: string,
  value: string,
  image?: string,
}

interface User {
  id: string,
  name: string,
  lastName: string,
  age: number,
  documents: Document[],
}

const user = createState<User>('user')
```

It is extremely important to specify the generic type whenever the second argument is not provided. In the code above,
we created a state that holds a `User`.

#### Declaring and reading local states
Local states are the only type of states that need to be declared. To do this, just assign the state to the component you wish to attach it to. See
the example below:

```tsx
import { NimbusJSX, createState } from '@zup-it/nimbus-backend-core'
import { Screen } from '@zup-it/nimbus-backend-express'
import { Column, Text } from '@zup-it/nimbus-backend-layout'

// (...) consider the context "user" declared in the previous example

const MyScreen: Screen = () => (
  <Column context={user}>
    <Text>Welcome {user.get('name')} {user.get('lastName')}!</Text>
  </Column>
)
```

In the code above we declared `user` to the root component, i.e. it will be accessible by every component in the screen.
After declaring the state, we printed the user name using the state api.

Notice that we don't need to use expression strings when using the Nimbus backend for Typescript lib. A State is an object that can be used directly
in the UI structure and will be serialized into expressions when needed. This is important so we can have type-safety when using states and
expressions, which is something we can't have in plain JSON.

> Important: when using the Nimbus Backend for Typescript, the developer never needs to write expressions, the state API is enough and is type-safe!

> Note: If a local state is not declared, when running the code in the frontend, Nimbus will be unable to find it, producing log messages like
`Couldn't find state of name "user"`.

### Local states in the frontend
If you're using Nimbus Compose and Nimbus SwiftUI in the frontend, it takes care of the state for you. You don't need to worry about it! The lib
processes the json and calculates the state values before sending data to the UI layer that renders the components.

## Implicit states
These are states that don't need to be declared in the JSON they're implicit to their context and are created by the frontend application. For
instance, a component for a text input may have the property "onChange", which is an event. The actions associated to this event must know what the
new value is. So we make the onChange event create a state in the frontend and provide it to the Nimbus context.

Every time an event has an implicit state, this state must have the same name as the event, in this case, the new value of the input can be retrieved
by accessing the value of the state "onChange". Via expression: `"@{onChange}"`.

These states are also found in [Actions](/action.md). The action to send a request, for instance, has the event `onSuccess` that must know the
response data to run the actions. When providing the actions for `onSuccess`, we can access the request data via expressions like:
`"@{onSuccess.status}"` or `"@{onSuccess.data.property}"`.

Implicit states are also used to create templates in the structural component `forEach`. Inside the template the current iteration can be accessed
via the expressions `"@{item}"` and `"@{index}"`. This is the only component that declares a implicit state for now, but more could be added in the

### Implicit states in the backend for Typescript
#### Using events with implicit contexts
It's very simple to use implicit states in the backend for typescript. See the examples below:

1. A Text Input:
```tsx
import { NimbusJSX, createContext } from '@zup-it/nimbus-backend-core'
import { log } from '@zup-it/nimbus-backend-core/actions'
import { TextInput } from '@zup-it/nimbus-backend-material'
import { Screen } from '@zup-it/nimbus-backend-express'

const MyScreen: Screen = () => <TextInput onChange={value => log(value)} />
```

The `onChange` property accepts a function that injects the implicit state for us (`value`). The UI created with the code above logs every text
typed into the input.

2. Sending a request and using the response:
```typescript
import { log, sendRequest } from '@zup-it/nimbus-backend-core/actions'
import { Product } from '../model/Product' // detailing this interface is not important for this example

const loadProducts = sendRequest<Product[]>({
  // ...
  onSuccess: response => log(`Products loaded: ${response.data.at(0).get('name')}`)
})
```

In the code above we use the `sendRequest` action to load a list of products and we receive an implicit state in
the `onSuccess` event. This implicit state contains the response returned by the server. We then log a message
telling the name of the first product in the list.

#### Creating components or actions with implicit contexts
Implicit contexts are always created by components or actions in the frontend, all we do
here is create references to them. If you're creating a component that has an event that spawns an implicit context, you
can use the function `createStateNode` to create the reference.

We use `createStateNode` instead of `createState` because we are not declaring a new state, they don't have an initial value and they are not
associated to the attribute `state` of a Component.

`createStateNode` accepts a single argument: the state's path (id). See the example below:

```tsx
import { NimbusJSX, FC, createStateNode } from '@zup-it/beagle-backend-core'

interface Props {
  onChange: (value: State<string>) => Actions,
}

const TextInput: FC<Props> = ({ id, onChange }) => {
  const onChangeActions = onChange(createStateNode('onChange'))
  return <component namespace="myApp" name="textInput" id={id} properties={{ onChange: onChangeActions }} />
}
```

First, we declared that our component accepts the attribute `onChange`, which should be a function that receives a
State of type `string` and returns `Actions` (an `Action` or an array of `Action`).

When we create the actual component via the tag `<component />`, we must execute `onChange` first to compute the actual
actions that should go into the component. A component can accept Nimbus Actions, but it can never receive functions as
its properties (functions are not serializable).

To create the implicit state that goes in the function argument, we use the function `createStateNode`. The single
argument we pass is the name of the state, this should be the same name the frontend uses. To avoid confusion, an implicit state spawned by an
event must always be identified by the event name. For instance, if the event name is `onChange`, the state id should be `onChange`.

The same strategy can be used when creating actions that spawn implicit states.

### Implicit states in Nimbus Compose
It's very simple to make an event create an implicit state in Nimbus Compose. See the example below for a component that renders a text input.

TODO: create and explain the example. Just pass the current value to the function received in the `onChange` attribute.

Notice that the component doesn't become coupled to Nimbus. This is exactly what you would do in most other scenarios. The Nimbus lib is responsible
for providing such function. See how the deserializer for this component work:

TODO: example of deserializer for the component

### Implicit states in Nimbus SwiftUI
It's very simple to make an event create an implicit state in Nimbus SwiftUI. See the example below for a component that renders a text input.

TODO: create and explain the example. Just pass the current value to the function received in the `onChange` attribute.

Notice that the component doesn't become coupled to Nimbus. This is exactly what you would do in most other scenarios. The Nimbus lib is responsible
for providing such function. See how the deserializer for this component work:

TODO: example of deserializer for the component

## Global State
The Global State can be used to share data between screens. Each Nimbus instance has a single Global State and it can be altered via both the backend
and the frontend.

To change the global state value via the backend (json), just use the state id "global". See the section
["Manipulating State Values"](#Manipulating-State-Values) for more details on changing state values.

> Attention: by giving the id "global" to a local state, you end up shadowing the global state, i.e. the global state becomes unreachable for that
node and its descendants.

### Global State in the backend for Typescript
To get a reference to the Global State, we must use the function `getGlobalState`. It is extremely important to type
this structure, and because of this, we recommend creating a single separated file for the Global State of your
application. See the example below:

`global-state.ts`:
```typescript
import { getGlobalState } from '@zup-it/nimbus-backend-core'
import { Product } from '../model/product' // detailing this interface is not important for this example
import { User } from '../model/user' // detailing this interface is not important for this example

export interface GlobalState {
  cart: Product[],
  user: User,
}

export const globalState = getGlobalState<GlobalState>()
```

`my-screen.ts`:
```tsx
import { NimbusJSX } from '@zup-it/nimbus-backend-core'
import { Screen } from '@zup-it/nimbus-backend-express'
import { globalState } from '../global-state'

export const MyScreen: Screen = () => <>Hello {globalState.get('user').get('name')}!</>
```

The Global State is the only type of Nimbus State that can be both read and written outside Beagle. Read the next section for more details.

### Global State in Nimbus Compose
TODO: check this text
To access the Global State in Nimbus Compose you must get use your nimbus instance:

```kotlin
val nimbus = Nimbus(baseUrl = "https://my-backend.com")
val globalState = nimbus.globalState
```

With a reference to the Global state, you can:
- read it: `globalState.getValueCopy()`. Gets a copy of the current state value.
- write to it: `globalState.set(newValue, path)`. Sets the current state value at the specified path. If no path is provided, the entire state is
replaced by `newValue`. To set user's name, for instance: `globalState.set("John", "user.name")`. If a path doesn't exist in the global context, it's
created.
- listen to changes: `onChange(listener)`. Subscribes to changes in the global state. `listener` must be a function that receives nothing and returns
nothing. The `onChange` function returns another function that, when called, removes the listener from the state.

### Global State in SwiftUI
TODO: adapt this text for Nimbus SwiftUI.
To access the Global State in Nimbus Compose you must get use your nimbus instance:

```kotlin
val nimbus = Nimbus(baseUrl = "https://my-backend.com")
val globalState = nimbus.globalState
```

With a reference to the Global state, you can:
- read it: `globalState.getValueCopy()`. Gets a copy of the current state value.
- write to it: `globalState.set(newValue, path)`. Sets the current state value at the specified path. If no path is provided, the entire state is
replaced by `newValue`. To set user's name, for instance: `globalState.set("John", "user.name")`. If a path doesn't exist in the global context, it's
created.
- listen to changes: `onChange(listener)`. Subscribes to changes in the global state. `listener` must be a function that receives nothing and returns
nothing. The `onChange` function returns another function that, when called, removes the listener from the state.

## Manipulating State Values from the backend
To change a state value, we must use the [action](/action.md) `setState`.

`setState` accepts a path and the new value. See the example below:

```json
{
  "_:component": "layout:column",
  "state": {
    "id": "user",
    "value": {
      "name": "John",
      "age": 30
    }
  },
  "children": [
    {
      "_:component": "material:button",
      "properties": {
        "text": "Click to change the user name to Joseph",
        "onPress": [{
          "_:action": "setState",
          "path": "user.name",
          "value": "Joseph"
        }]
      }
    }
  ] 
}
```

The UI above renders a button that when clicked changes the state `user.name` from "John" to "Joseph".

Any state, local, implicit or global can have its value changed by the `setState` action.

### Changing state values in the backend for Typescript
In the backend for typescript, you shouldn't use the `setState` action directly. Instead, you should use the method `set` of a State object. When
the lib is serializing the UI tree, this will be converted to the `setState` action. See the example below:

```tsx
const user = createState<User>('user', {
  name: 'John',
  age: 30,
})

const MyScreen: Screen = () => (
  <Column state={user}>
    <Button onPress={user.get('name').set('Joseph')}>Click to change the user name to Joseph</Button>
  </Column>
)
```
