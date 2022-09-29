# Actions
Check the [specification](/specification/action.md) to know more about the definition of Nimbus Actions.

## Creating your own Actions
Nimbus already comes with some [pre-defined actions](/specification/default-actions.md), but for most applications, they're not enough and we
need to extend the set of actions.

We'll create an action that logs a message to the console as an example. This action receives a single parameter:

- `message`: string, required. The message to log.

### 1. Write the Action Handler
```kotlin
fun log(message: String) {
    print(message)
}
```

First, we get the property "message" from the event. Then, we check if it is correct (if it is a String) and print it to the console.

An `ActionTriggeredEvent` has the following properties:

- `action`: the action triggered by the event. Use this object to find the name of the action, its properties and metadata. This will be the only
information you'll need for almost every action handler you'll write.
- `scope`: the scope where the action was triggered. This will always be a `ServerDrivenEvent` and contain information like the event name and the
entire scope hierarchy. Example of scope hierarchy: this event > parent event > node > another node > view > nimbus. Any of these scopes can be
retrieved via the `parent` pointer.
- `dependencies`: tell the Nimbus lifecycle which dependencies may have changed so it can propagate updates through its dependency graph. You'll
probably never need to use this, but it's used by the core action `setState` to update the UI after a state changes its value.

For most action handlers, we'll only need `event.action.properties`.

### 2. Register the action handler
The same structure used for registering components and actions is used for registering operations: NimbusComposeUILibrary. See the example below:

```kotlin
val myAppUI = NimbusComposeUILibrary("myApp")
    .addAction("log") {
        val message = it.action.properties?.get("message")
        if (message is String) log(message)
    }
)

val customActions: Map<String, ActionHandler> = mapOf(
    "myApp:log" to { log(it) }
)
```
Whenever the action "myApp:log" is used by a server driven view, the function `log` will be called with the event.

Attention: we intend to create a more intuitive and type-safe way for writing actions.

### 3. Certify your ui lib is provided in the configuration
```kotlin
private val nimbus = Nimbus(
    // ...
    ui = listOf(myAppUI),
)
```

# Read next
:point_right: [Configuration](configuration.md)
