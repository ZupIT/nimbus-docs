# Operations
Operations, along with [State](/state) and [Actions](/action) are what make dynamic content possible in Nimbus.

Operations are pure functions that receive a set of arguments and must return a value based on the arguments they receive. They're used to
transform a state value.

Let's say we want to print a string, but we need to capitalize all letters. Considering the value we want is in the state `user.name`, instead of
writing the expression `"@{user.name}"`, we can use an operation: ``"@{uppercase(user.name)}"``. The operation `uppercase` transforms all letters
in a string to uppercase.

Operations are always used inside expressions and they must follow the pattern `name_of_the_operation(argument1, argument2, ...)`. The name of the
operation is the string that identifies the function to use in the frontend and it must be comprised of only letters, numbers and underline ("_"); it
can't start with a number; and it can't be either "true", "false" or "null". In other words, the operation name must match the regex
`/^(?!(true$|false$|null$))([a-z]|[A-Z]|_)\w*$/`.

An operation argument can be:
- A state path. Examples: `myState`, `user.name`, `global.cart.length`.
- A number. Examples: `10`, `20.589`.
- A boolean. Examples: `true`, `false`.
- A string. Examples: `'Hello World'`, `'This is a string'`. Notice that strings must be put inside single quotation marks.
- Another operation: Examples: `sum(user.age, 1)`, `gt(sum(user.age, 5), 18)`.

Arguments cannot be maps (`{ key: value }`) or arrays (`[1, 2]`). To use a map or array as an argument, it must be stored in a state first.

# Example
A very simple example for the use of operations is a screen with a counter. The counter is incremented whenever the user presses a button.

```json
{
  "_:component": "layout:column",
  "state": {
    "counter": 0
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

# Default operations
Nimbus ships with some basic operations, you can check them [here](/default-operations.md).

# Custom operations
Just like [components](/component.md) and [actions](/action.md), Nimbus must provide a way for the developer to register new operations. Custom
operations are very useful for doing custom formatting like currency, phone numbers and documents; applying filters to an array; mapping contents of
an array or object; etc.

How a new operation is registered for Nimbus is particular to each implementation. Please check the platform-specific documentation to know more.

# Read next
:point_right: [Default operations](/default-operations.md)
