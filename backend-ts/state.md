> Article level: intermediary.

# State
To check the definition of a State in Nimbus and its purposes, please check the [specification](specification/state).

# Local states
## Creating a local state
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

## Declaring and reading local states
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

# Implicit states
## Using events with implicit states
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

## Creating components or actions with implicit contexts
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

# Global State
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

# Manipulating State Values from the backend
To change a state value, we must use the [action](/action.md) `setState`, but in the backend for typescript, you shouldn't use the `setState` action directly. Instead, you should use the method `set` of a State object, this returns an instance of the `setState` action and is much easier to use. 
See the example below:

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
