> Article level: intermediary.

# State
To check the definition of a State in Nimbus and its purposes, please check the [specification](specification/state).

# Local states
Nimbus compose takes care of the local states for you. You don't need to worry about it! The lib processes the json and calculates the state values
before sending data to the UI layer that renders the components.

# Implicit states
It's very simple to make an event create an implicit state in Nimbus Compose. See the example below for a component that renders a text input.

```kotlin
@Composable
fun NimbusTextField(text: String, onChange: (Any?) -> Unit) {
    TextField(value = text, onValueChange = {
        onChange(it)
    })
}
```

Notice that the component doesn't become coupled to Nimbus. This is exactly what you would do in most other scenarios. The Nimbus lib is responsible
for providing such function. See how the deserializer for this component work:

```kotlin
val customComponents: Map<String, @Composable ComponentHandler> = mapOf(
    "material:textField" to @Composable { element, _ , _ ->
        NimbusTextField(
            text = element.properties?.get("text") as String,
            onChange = element.properties!!["onChange"] as (Any?) -> Unit,
        )
    }
)
```

## Global State
To access the Global State in Nimbus Compose you must get use your nimbus instance:

```kotlin
Nimbus(config = config) {
val globalState = nimbusAppState.config.core.globalState
```

With a reference to the Global state, you can:
- read it: `globalState.getValueCopy()`. Gets a copy of the current state value.
- write to it: `globalState.set(newValue, path)`. Sets the current state value at the specified path. If no path is provided, the entire state is
replaced by `newValue`. To set user's name, for instance: `globalState.set("John", "user.name")`. If a path doesn't exist in the global context, it's
created.
- listen to changes: `onChange(listener)`. Subscribes to changes in the global state. `listener` must be a function that receives nothing and returns
nothing. The `onChange` function returns another function that, when called, removes the listener from the state.
