# Actions
Check the [specification](/specification/action.md) to know more about the definition of Nimbus Actions.

## Creating your own Actions
Nimbus already comes with some [pre-defined actions](/specification/default-actions.md), but for most applications, they're not enough and we
need to extend the set of actions.

We'll create an action that logs a message to the console as an example. This action receives two parameters:

- `message`: string, required. The message to log.
- `level`: LogLevel, optional. The type of log.

### 1. Write the Action Handler
Write the function like any other Kotlin function and add the annotation `@AutoDeserialize` to it.

```kotlin
import br.com.zup.nimbus.annotation.AutoDeserialize

enum class LogLevel { Info, Warning, Error }

@AutoDeserialize
fun log(message: String, level: LogLevel?) {
    println("${level ?: LogLevel.Info}: $message")
}
```

`@AutoDeserialize` marks your action handler (function) for code generation. The next time you build the project, a version of this composable 
function that accepts an instance of `ActionTriggeredEvent` will be available.

To know more about the auto-deserialization, [read this topic](auto-deserialization.md).

### 2. Register the action handler
The same structure used for registering components and operations is used for registering actions: `NimbusComposeUILibrary`. See the example below:

```kotlin
val myAppUI = NimbusComposeUILibrary("myApp")
    .addAction("log") { log(it) }
)
```

> Attention: this code will be marked as invalid by the IDE until a build is performed (code generation).

Whenever the action "myApp:log" is used by a server driven view, the function `log` will be called.

### 3. Register the UI Library to Nimbus
If the UI Library is not yet registered to Nimbus, please do so:

```kotlin
private val nimbus = Nimbus(
    baseUrl = BASE_URL,
    ui = listOf(layoutUI, myAppUI),
)
```

## Manual vs Automatic Action deserialization
When registering an action handler, Nimbus provides all the data that came from the JSON so you can call the action handler, this data is wrapped in 
the type `ActionTriggeredEvent` and the properties of the action can be accessed via `ActionTriggeredEvent#action.properties`, which is a map of 
`String` to `Any?`.

Deserializing items in a list of unknown types can be very boring, repetitive and dangerous. For this reason, we recommend using the method described
in this topic (automatic), which requires both the packages "Nimbus Compose Annotations" and "Nimbus Compose Processor".

If you want to learn the manual approach, which doesn't rely on code generation, please read the topic 
["Actions: manual deserialization"](manual/action.md) instead.

# Read next
:point_right: If you want to learn how to manually deserialize an operation: [Actions: manual deserialization](manual/action.md)
:point_right: If you want to know more about the auto-deserialization: [Automatic deserialization](auto-deserialization.md)
:point_right: Otherwise: [Configuration](configuration.md)
