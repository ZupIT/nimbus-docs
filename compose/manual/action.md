# Actions (manual deserialization)
This topic will show how to manually deserialize an action handler. If you want to learn the automatic and recommended approach (code generation),
please read the topic ["Actions"](../action.md) instead.

We'll use the same action implemented [here](../action.md#creating-your-own-actions).

You'll implement the action handler just like the automatic version, but without the annotation `@AutoDeserialize`.

## Register the action handler deserializer to the UI Library
For manual deserialization, we need to use the properties map at `ActionTriggeredEvent#action.properties`. An instance of `ActionTriggeredEvent` is 
provided as the parameter of the function used to register the action handler. See the example below:

```kotlin
import br.zup.com.nimbus.compose.ui.NimbusComposeUILibrary

val myAppUI = NimbusComposeUILibrary("myApp")
    .addAction("log") {
        val message = it.action.properties?.get("message")
        val levelString = it.action.properties?.get("level")
        val level = LogLevel.values().find() { it.name == levelString } ?: LogLevel.Info
        if (message is String) log(message, levelString)
    }
)
```

Whenever the action "myApp:log" is used by a server driven view, the function registered above will be called with the `ActionTriggeredEvent` 
representing it.

The code above is just an example, although it is safe to use, it doesn't give any error message in case a required property is invalid or absent. 
To help with the deserialization, Nimbus provides the class `AnyServerDrivenData`, see below an example of how to use it:

```kotlin
import br.zup.com.nimbus.compose.ui.NimbusComposeUILibrary
import com.zup.nimbus.core.deserialization.AnyServerDrivenData

val myAppUI = NimbusComposeUILibrary("myApp")
    .addAction("log") {
        val data = AnyServerDrivenData(it.action.properties)
        val message = data.get("message").asString()
        val level = data.get("level").asEnum(LogLevel.values())
        if (data.hasError()) throw IllegalArgumentException(data.errorsAsString())
        log(message, levelString)
    }
)
```

`AnyServerDrivenData` will collect the deserialization errors if they happen, without stopping the execution. It also makes automatic type coercions
for helping with type management. If you are going to use manual deserialization, we highly recommend the use of this class.

If we find any deserialization error, instead of calling the action handler, we throw an exception, which will be treated and printed as an error
log by Nimbus.

### `ActionTriggeredEvent`
An `ActionTriggeredEvent` has the following properties:

- `action`: the action triggered by the event. Use this object to find the name of the action, its properties and metadata. This will be the only
information you'll need for almost every action handler you'll write.
- `scope`: the scope where the action was triggered. This will always be a `ServerDrivenEvent` and contain information like the event name and the
entire scope hierarchy. Example of scope hierarchy: this event > parent event > node > another node > view > nimbus. Any of these scopes can be
retrieved via the `parent` pointer.
- `dependencies`: tell the Nimbus lifecycle which dependencies may have changed so it can propagate updates through its dependency graph. You'll
probably never need to use this, but it's used by the core action `setState` to update the UI after a state changes its value.

Most action handlers will only need `action.properties`.

# Read next
:point_right: [Configuration](configuration.md)
