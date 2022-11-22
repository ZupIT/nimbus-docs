# Components (manual deserialization)
Check the [specification](/specification/component.md) to know more about the definition of Nimbus Components.

This topic will show how to manually deserialize a component. If you want to learn the automatic and recommended approach (code generation), please
read the topic ["Components"](../component.md) instead.

We'll use the same component implemented [here](../component.md#creating-components).

You'll implement the component just like any composable function, without the annotation `@AutoDeserialize` of the automatic process.

## Register the component deserializer to the UI Library
For manual deserialization, we need to use the properties map at `ComponentData#node.properties`. An instance of `ComponentData` is provided as the
parameter of the function used to register the component. See the example below:

```kotlin
import br.zup.com.nimbus.compose.ui.NimbusComposeUILibrary

val myAppUI = NimbusComposeUILibrary("myApp")
    .addComponent("button") @Composable {
        val onPress = it.node.properties?["onPress"]
        MyButton(
            text = it.node.properties?.get("text") as String,
            onPress = if (onPress is ServerDrivenEvent) { onPress.run() } else {},
            enabled = element.properties?["enabled"] as? Boolean,
        )
    },
)
```

Whenever the component "myApp:button" is used by a server driven view, the composable function above will be called with the `ComponentData`
representing it.

The code above is just an example, it is still unsafe since we're not always checking if the value is of the type we expect before casting it. We also
have no treatment for when the expected properties are not available. To help with this, Nimbus provides the class `AnyServerDrivenData`, see below
an example of how to use it:

```kotlin
import br.zup.com.nimbus.compose.ui.NimbusComposeUILibrary
import com.zup.nimbus.core.deserialization.AnyServerDrivenData

val myAppUI = NimbusComposeUILibrary("myApp")
    .addComponent("button") @Composable {
        val data = AnyServerDrivenData(it.node.properties)
        val text = data.get("text").asString()
        val onPress = data.get("onPress").asEvent()
        val enabled = data.get("enabled").asBooleanOrNull()
        if (data.hasError()) {
            println(data.errorsAsString())
            Text("Deserialization Error")
        } else {
            MyButton(
                text = text,
                onPress = { onPress.run() },
                enabled = enabled,
            )
        }
    },
)
```

`AnyServerDrivenData` never throws, since `try - catch` blocks can't be used inside composable functions. It also makes automatic type coercions for
helping with type management. If you are going to use manual deserialization, we highly recommend the use of this class.

Whenever a property represents an event, it will be an instance of `ServerDrivenEvent`, which can be run through the function `run()`. Another way
to run an event is through `run(Any)`, which can be used for events that create a state, like an `onChange` inside a text input.

### `ComponentData`

A component builder (last parameter of `addComponent`) accepts a single argument which is of type `ComponentData`. A `ComponentData` contains:

- `children`: a composable function to render the children (content) of this component.
- `childrenAsList`: a function that returns a list of composable functions, each one representing a child. This exists to give support for components
like the `LazyRow` and `LazyColumn`, you'll probably never have to use it.
- `node`: the current component.
  - `id`: the unique id for this component;
  - `component`: the name of this component. In this example, it will always be "myApp:button";
  - `properties`: the map of properties for this component;
  - `children`: a collection of `ServerDrivenNode` representing the children of this component.

# Read next
:point_right: [State](state.md)
