> Article level: intermediary.

# State
Nimbus doesn't have an embedded script language, and we defend that it shouldn't have one. But we still need to make it
possible to create dynamic UIs from the backend. To solve this issue, Nimbus introduces three concepts: State,
[Operations](/operation.md) and [Actions](/action.md). This topic talks about States.

States are the Nimbus way of creating and manipulating a value that can shared across multiple components. We can make an
equivalence to variables in programming languages, they can be declared (local states), some are global (global
state) and others are injected into your code (implicit states).

The UI tree responds to every modification of a state value, it's dynamic!

**Important**: States are variables of Nimbus, they're not variables of your backend application, i.e. Nimbus States are resolved
when the code is run by the frontend, and not when the request is made to the server.

# Local states
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

# Implicit states
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
future.

A simple example for this is a screen with a text input and a paragraph below it. The paragraph needs to show the exact content typed into the text
input. See a solution below:
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

## Global State
The Global State can be used to share data between screens. Each Nimbus instance has a single Global State and it can be altered via both the backend
and the frontend.

To change the global state value via the backend (json), just use the state id "global". See the section
["Manipulating State Values"](#Manipulating-State-Values) for more details on changing state values.

> Attention: by giving the id "global" to a local state, you end up shadowing the global state, i.e. the global state becomes unreachable for that
node and its descendants.

Example:
```json
{
  "_:component": "layout:text",
  "properties": {
    "text": "@{global.user.name}"
  }
}
```

As long as the frontend sets the value for `user.name` in the Global Context, the code above prints the user's name.

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
