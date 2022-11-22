# Components
Check the [specification](/specification/component.md) to know more about the definition of Nimbus Components.

## Creating components
Let's see an example for creating a Button component. This component must accept the following properties:

- `text`: string, required. The text to show inside the button, i.e. its label.
- `enabled`: boolean, optional. Whether the button is clickable (true) or not (false). Default is true.
- `onPress`: function, required. Callback to run when the button is pressed.

### 1. Create the composable function
Create your component just like any other Composable and add the annotation `@AutoDeserialize`:

```kotlin
import br.com.zup.nimbus.annotation.AutoDeserialize

@Composable
@AutoDeserialize
fun MyButton(text: String, enabled: Boolean?, onPress: () -> Unit) {
    Button(enabled = enabled != false, content = { Text(text) }, onClick = { onPress() })
}
```

`@AutoDeserialize` marks your composable function for code generation. The next time you build the project, a version of this composable function
that accepts an instance of `ComponentData` will be available.

To know more about the auto-deserialization, [read this topic](auto-deserialization.md).

### 2. Register the component to the UI Library
The same structure used for registering actions and operations is used for registering components: `NimbusComposeUILibrary`. See the example below:

```kotlin
val myAppUI = NimbusComposeUILibrary("myApp")
    .addComponent("button") @Composable { MyButton(it) },
)
```

> Attention: this code will be marked as invalid by the IDE until a build is performed (code generation).

Whenever the component "myApp:button" is used by a server driven view, the composable function above will be called.

### 3. Register the UI Library to Nimbus
If the UI Library is not yet registered to Nimbus, please do so:

```kotlin
private val nimbus = Nimbus(
    baseUrl = BASE_URL,
    ui = listOf(layoutUI, myAppUI),
)
```

## Manual vs Automatic Component deserialization
When registering a component, Nimbus provides all the data that came from the JSON so you can call the composable function, this data is wrapped in 
the type `ComponentData` and the properties of the component can be accessed via `ComponentData#node.properties`, which is a map of `String` to
`Any?`.

Deserializing items in a list of unknown types can be very boring, repetitive and dangerous. For this reason, we recommend using the method described
in this topic (automatic), which requires both the packages "Nimbus Compose Annotations" and "Nimbus Compose Processor".

If you want to learn the manual approach, which doesn't rely on code generation, please read the topic 
["Components: manual deserialization"](manual/component.md) instead.

# Read next
:point_right: If you want to learn how to manually deserialize a component: [Components: manual deserialization](manual/component.md)
:point_right: If you want to know more about the auto-deserialization: [Auto component deserialization](auto-deserialization.md)
:point_right: Otherwise: [State](state.md)
