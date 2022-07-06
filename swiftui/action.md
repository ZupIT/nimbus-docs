# Actions
Check the [specification](/specification/action.md) to know more about the definition of Nimbus Actions.

## Creating your own Actions
Nimbus already comes with some [pre-defined actions](/specification/default-actions.md), but for most applications, they're not enough and we
need to extend the set of actions.

We'll create an action that logs a message to the console as an example. This action receives a single parameter:

- `message`: string, required. The message to log.

### 1. Write the Action Handler
```swift
let log: Action = { event in
  if let message = event.action.properties?["message"] as? String {
    print(message)
  }  
  return nil
}
```

We get the property "message" from the event, cast to expected type `String` and print it to the console if message is not nil.

`Action` is a typealias for `(ActionEvent) -> KotlinUnit?`. `KotlinUnit?` represents void and we should return nil.
An `ActionEvent` has the following properties:

- `action`: the action triggered by the event. Use this object to find the name of the action, its properties and metadata.
- `name`: the name of the event that triggered the action, i.e. the key of the node property that declared it. Example: a component "Button" triggers
the action found in the property "onPress" when it's pressed. "onPress" is the "name" of this event.
- `node`: the node (component) containing the action.
- `view`: the view (`ServerDrivenView`) containing the node that declared the action.

For most action handlers, we'll only need `event.action.properties`.

### 2. Register the action handler
You must create the structure that maps the action name to its handler. This is the map we pass in the function `actions` when configuring the `Nimbus` entrypoint. 
See the example below:

```swift
let actions: [String: Action] = ["myApp:log": log]
```

Whenever the action "myApp:log" is used by a server driven view, the function `log` will be called with the event. 

### 3. Certify your action map is configured in the Nimbus entrypoint
```swift
Nimbus(baseUrl: "base") {
  // Your navigator here
}
// another configurations as .components(components) here
.actions(actions)
```

# Read next
:point_right: [Configuration](configuration.md)


# Read next
:point_right: [TODO](/todo_link)