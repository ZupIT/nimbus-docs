# Actions
Check the [specification](/specification/action.md) to know more about the definition of Nimbus Actions.

## Creating your own Actions
Nimbus already comes with some [pre-defined actions](/specification/default-actions.md), but for most applications, they're not enough and we
need to extend the set of actions.

We'll create an action that logs a message to the console as an example. This action receives a single parameter:

- `message`: string, required. The message to log.

### 1. Write the Action Handler
```kotlin
fun log(event: ActionEvent) {
    val message = event.action.properties?.get("message")
    if (message is String) print(message)
}
```

First, we get the property "message" from the event. Then, we check if it is correct (if it is a String) and print it to the console.

An `ActionEvent` has the following properties:

- `action`: the action triggered by the event. Use this object to find the name of the action, its properties and metadata.
- `name`: the name of the event that triggered the action, i.e. the key of the node property that declared it. Example: a component "Button" triggers
the action found in the property "onPress" when it's pressed. "onPress" is the "name" of this event.
- `node`: the node (component) containing the action.
- `view`: the view (`ServerDrivenView`) containing the node that declared the action.

For most action handlers, we'll only need `event.action.properties`.

### 2. Register the action handler
You must create the structure that maps the action name to its handler. This is the map we pass in the parameter `actions` when creating
an instance of Nimbus. See the example below:

```kotlin
val customActions: Map<String, ActionHandler> = mapOf(
    "myApp:log" to { log(it) }
)
```

Whenever the action "myApp:log" is used by a server driven view, the function `log` will be called with the event. 

### 3. Certify your action map is provided in the configuration
```kotlin
private val nimbus = Nimbus(
    baseUrl = BASE_URL,
    actions = customActions,
)
```
