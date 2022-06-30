> Article level: basic.

# Components
Check the [specification](/specification/component.md) to know more about the definition of Nimbus Components. 

# Creating components

### First step
You can define a custom component that maps to a composable NimbusTextField like below
```kotlin
val customComponents: Map<String, @Composable ComponentHandler> = mapOf(
    "material:textField" to @Composable { element, _ , _ ->
        NimbusTextField(
            text = element.properties?.get("text") as String,
            onChange = element.properties!!["onChange"] as (Any?) -> Unit,
        )
    },
    "material:button" to @Composable { element, _ , _ ->
        NimbusButton(
            text = element.properties?.get("text") as String,
            onPress = element.properties!!["onPress"] as (Any?) -> Unit,
            enabled = element.properties!!["enabled"] as Boolean ?: true,
        )
    },
)
```

### Second step
```kotlin
@Composable
fun NimbusTextField(text: String, onChange: (Any?) -> Unit) {
    TextField(value = text, onValueChange = {
        onChange(it)
    })
}

@Composable
fun NimbusButton(text: String, enabled: Boolean = true, onPress: (Any?) -> Unit) {
    Button(enabled = enabled, content = { Text(text) }, onClick = { onPress(null) })
}
````
### Third step
You must serve the map to nimbus config, also you can provide several maps like below
```kotlin
private val config = NimbusConfig(
        baseUrl = BASE_URL,
        components = customComponents + layoutComponents,
    )
```

# Read next
:point_right: [Action](/action)
