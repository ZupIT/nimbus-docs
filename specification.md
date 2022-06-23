> Article level: advanced.

# Structure
Nimbus needs to be able to serialize and deserialize the UI in order to achieve the desired behavior of driving the user interface via a server-side
application. The format used for such serialization is JSON.

As most UI representations, Nimbus use a tree to represent the interface. Each node in this tree is a UI component with a name, id, properties and
children.

## Component (Node)
A Node or Component in the Nimbus UI Tree follows the format below:

> Attention: given its simplicity, we're going to use the typescript syntax to define the json structure.

```typescript
interface Node {
  '_:component': string,
  id?: string,
  properties?: Record<string, any>, // Record is equivalent to map.
  state?: {
    id: string,
    value: any,
  },
  children?: Node[],
}
```

The only required property of a node is "\_:component". Notice the very weird prefix "_:". This prefix is reserved to Nimbus keys in the JSON. It is
weird so it doesn't clash with application specific keys. Never use this prefix for keys in your application, since every time we need a new reserved
key we'll use it and we won't consider it to be a breaking change.

the key "\_:component" identifies the object as being a Nimbus Node and its value tells which UI component must be rendered. See the example below:

```json
{
  "_:component": "layout:column",
  "children": [
    {
      "_:component": "layout:text",
      "properties": {
        "text": "Hello"
      },
    },
    {
      "_:component": "layout:text",
      "properties": {
        "text": "World"
      }
    }
  ]
}
```

The JSON above renders a column with two texts: "Hello" in the first line and "World" in the second line. It's important to notice that these
components are part of the library "Nimbus Layout" and are not included in any Nimbus implementation. The intention here is just to show an example of
a JSON.

The properties of a component are:
- `_:component`: required. The name that identifies the component. It must follow the format: namespace:name. The only components that don't need a
namespace are the default components that come with any Nimbus implementation. Examples: if, forEach, layout:text, material:button, layout:column.
- `id`: a unique id for the node (considering the tree). When not specified, a random id is assigned by the frontend.
- `properties`:  a map of properties for the component. The structure of this map will vary depending on the value of `_:component`. A Text, for
instance, will have a completely different set of properties when compared to a Row, Colum or TabBar.
- `state`: declares a state visible to this node and its descendants. See the section "State" in this article.
- `children`: the nodes that are children of this node. When absent, null or an empty array, this node is a leaf.
A State has the following structure:

To know more about components, please check the [dedicated topic](/component.md).

The Nimbus specification already declares 2 structural components, i.e. any implementation must have them. Check [here](/default-components.md) all
default components.

## Action
A component property may be a list of [actions]. Actions are side-effects executed by the frontend when a given event occurs. For instance, when a
button is pressed (event) we might want to navigate to another screen (action). Actions are identified by the key `_:action` and its value must
identify which action to run. See the format of an Action object below:

```typescript
interface Action {
  '_:action': string,
  properties?: Record<string, any>,
  metadata?: Record<string, any>,
}
```

Just like the components, actions have a single required parameter: `_:action`. The properties are:
- `_:action`: required. The name that identifies the action. It must follow the format: namespace:name. The only actions that don't need a
namespace are the default actions that come with any Nimbus implementation. Examples: sendRequest, log, setState, material:alert, material:confirm.
- `properties`: a map of properties for the action. The structure of this map will vary depending on the value of `_:action`. A log instruction, for
instance, will have a completely different set of properties when compared to a an instruction to send a request or change a state value.
- `metadata`: this is an advanced feature that allow us to pass data that won't be used by the action itself, but might be useful for logging data or
creating analytics records.

Below, you can see an example of an action in the JSON:

```json
{
  "_:component": "material:button",
  "properties": {
    "text": "click me",
    "onPress": [{
      "_:action": "log",
      "properties": {
        "message": "The button has been clicked!"
      }
    }]
  }
}
```

In the example above, a button is rendered and when it's pressed, it logs a message to the console. We say that "onPress" is the event name and its
value is the list of actions to execute when the event occurs. Notice that the value is an array. Event values must always be arrays, if a single
action must run, the array will a have a single Action.

To know more about actions, please check the [dedicated topic](/action.md).

The Nimbus specification already declares some basic actions, i.e. any implementation must have them. Check [here](/default-actions.md) all
default actions.

## Expression
Component properties or Action properties can be dynamic, i.e. they can vary according to a state. To use state values instead of hardcoded values,
we use expressions.

A simple example is a screen with a text input and a paragraph below it. If we say the paragraph needs to show the exact content typed into the
text input, then we must use state and expressions. See the examples below:

Hardcoded: the text won't change.
```json
{
  "_:component": "layout:column",
  "children": [
    {
      "_:component": "material:textInput",
      "properties": {
        "placeholder": "Type the text here",
        "value": "Hardcoded value"
      }
    },
    {
      "_:component": "layout:text",
      "properties": {
        "text": "Hardcoded value"
      }
    }
  ]
}
```

Dynamic: the text changes according to the input:
```json
{
  "_:component": "layout:column",
  "state": {
    "id": "text",
    "value": ""
  },
  "children": [
    {
      "_:component": "material:textInput",
      "properties": {
        "placeholder": "Type the text here",
        "value": "@{text}",
        "onChange": [{
          "_action": "setState",
          "path": "text",
          "value": "@{onChange}"
        }]
      }
    },
    {
      "_:component": "layout:text",
      "properties": {
        "text": "@{text}"
      }
    }
  ]
}
```

> Attention: this is just an example, none of these components actually exist in the base Nimbus library.

Expressions are everything between `@{` and `}` in a string. These marks can be scaped with the character `\` if necessary. Expressions can refer to
a state, part of a state or execute an operation. See the examples below:

- State: `"@{person}"`
- Part (member) of a state: `"@{person.name}"`
- Array access: `"@{person.documents[0]}"`
- Operation: `"gt(person.age, 18)"`

An expression must follow a very specific grammar. This grammar is not regular and can be recognized by a Push Down Automaton, i.e. it's a
context-free grammar. Knowing this is not necessary for using a Nimbus library, but it is a necessary for developing a new frontend lib. Check [this
article](/expression-grammar.md) to see the full grammar specification.

To know more about states, please read the [dedicated topic](/state).
To know more about operations, please read the [dedicated topic](/operation).

## State
Along with Expressions, States allow dynamic UIs to be declared in Nimbus. A State is visible to the node where it's created and all its descendants.

A State can be one of three types:

### Local states
A local state is declared in the JSON using the key "state" of a node (component). The Local State is an object with the following properties:

- `id`: required. A string to identify the state. It uses only the characters a to z, A to Z, _ and 0 to 9. This can't start with a number. In
other words, it must match the regex `/^([a-z]|[A-Z]|_)\w*$/`.
- `value`: optional. The initial value for the state. Can be any JSON type. Null by default.

### Implicit states
These are states that don't need to be declared in the JSON, they're implicit to their context. For instance, a component for a text input may have
the property "onChange", which is an event. The actions associated to this event must know what the new value is. Every time an event has an implicit
state, this state must have the same name as the event, in this case, the new value of the input can be accessed through the expression
`"@{onChange}"`. An example of this can be seen in the example for expressions in the previous topic.

Implicit states are also used to create templates in the structural component `forEach`. Inside the template the current iteration can be accessed
via the expressions `"@{item}"` and `"@{index}"`. This is the only component that declares a implicit state for now, but more could be added in the
future.

### Global states
Global states are those who survive the current screen lifecycle. They don't exist the JSON because they're declared in a structure above the current
screen. The only state with this behavior for now is the Global State, which is declared for the current Nimbus instance and is shared across all
screens withing this instance.

To know more about states, please check the [dedicated topic](/state.md).


## Operation
Operations are functions without side-effects, they receive a set of parameters and return a result, that's all. Operations must only be used inside
expressions and their purpose is to execute some calculation on top of a state value. See the example below:

```json
{
  "_:component": "layout:column",
  "state": {
    "id": "counter",
    "value": 0
  },
  "children": [
    {
      "_:component": "layout:text",
      "properties": {
        "text": "Current value: @{counter}",
      }
    },
    {
      "_:component": "material:button",
      "properties": {
        "text": "Add +1",
        "onPress": [{
          "_:action": "setState",
          "path": "counter",
          "value": "@{sum(counter, 1)}"
        }]
      }
    }
  ]
}
```

The example above uses an operation to change a state value based on the previous value.

The operation name identifies which function to run in the frontend, it must follow the same name scheme as the state id, i.e., it must have only the characters a to z, A to Z, _ and 0 to 9. And it can't start with a number. In other words, it must match the regex `/^([a-z]|[A-Z]|_)\w*$/`.

The Nimbus specification already declares some basic operations, i.e. any implementation must have them. Check [here](/default-operations.md) all
default operations.
