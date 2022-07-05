# Components
Check the [specification](/specification/component.md) to know more about the definition of Nimbus Components. 

## Creating components
Let's see an example for creating a Button component. This component must accept the following properties:

- `text`: string, required. The text to show inside the button, i.e. its label.
- `enabled`: boolean, optional. Whether the button is clickable (true) or not (false). Default is true.
- `onPress`: function, required. Callback to run when the button is pressed.

### 1. Create the composable function
Create your component just like any other Composable:

```kotlin
@Composable
fun MyButton(text: String, enabled: Boolean = true, onPress: () -> Unit) {
    Button(enabled = enabled, content = { Text(text) }, onClick = { onPress() })
}
```

### 2. Register the component to Nimbus
Tell Nimbus the component exists, its name, and how to deserialize a Nimbus UI node (`ServerDrivenNode`) into the component.

We must use the components map to do it, i.e. the map we provide to the parameter `components` when creating an instance of Nimbus.

```kotlin
val customComponents: Map<String, @Composable ComponentHandler> = mapOf(
    "myApp:button" to @Composable { element, _ , _ ->
        MyButton(
            text = element.properties?.get("text") as String,
            onPress = { (element.properties?["onPress"] as (Any?) -> Unit)(null) },
            enabled = element.properties?["enabled"] as Boolean ?: true,
        )
    },
)
```

Whenever the component "myApp:button" is used by a server driven view, the composable function above will be called with the `ServerDrivenNode`
representing it (element). 

In the example, we casted `onPress` to `(Any?) -> Unit`. This is because every property function that comes from Nimbus has this signature. We then
execute it passing `null`. This is so we can provide [implicit state](state.md#implicit-states) values to Nimbus. Most components won't need this and
they will pass `null`, just like `myApp:button`. But some components will need this feature: text inputs, for instance, must pass its current value to
the `onChange` function.

In most cases, we'll only use the first argument of a `ComponentHandler`, i.e. the node itself (`ServerDrivenNode`). A node represents a component and
it contains the following properties:

- `id`: the unique id for this component;
- `component`: the name of this component. In this example, it will always be "myApp:button";
- `properties`: the map of properties for this component;
- `children`: a collection of `ServerDrivenNode` representing the children of this component.

The other two variables received by a `ComponentHandler` are:

- children: `@Composable () -> Unit`. The children (composable) for this component. If there are no children, it renders nothing. Components that
don't accept children, like buttons, should not care about this.
- parentElement: `ServerDrivenNode?`. The parent `ServerDrivenNode`. `null` if this is the root node. In most cases, this is not needed.

> The component deserialization is something we're not happy with. Expect changes before a stable release.

### 3. Certify your component map is provided in the configuration
```kotlin
private val nimbus = Nimbus(
        baseUrl = BASE_URL,
        components = customComponents + layoutComponents,
    )
```

## Read next
:point_right: [State](state.md)
