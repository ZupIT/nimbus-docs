> Article level: basic.

# Creating actions
Check the [action specification](/specification/action.md) to know more about the definition of Nimbus Actions. 

### First step
Here you define the custom action map. In this example we define that "alert" action is implemented by the showAlert function
```kotlin
val customActions: Map<String, ActionHandler> = mapOf(
    "alert" to { event: ActionEvent ->
        showAlert(event)
    }
)
```

### Second step

```kotlin
fun showAlert(event: ActionEvent) {
    val text = event.action.properties!!["text"] as String?
    NimbusTheme.nimbusStaticState?.let {
        Toast.makeText(it.applicationContext, text, Toast.LENGTH_SHORT).show()
    }
}
```
### Third step
Then when your create the nimbus config, you can concatenate with the components map like below
```kotlin

private val nimbus = Nimbus(
    actions = layoutActions + customActions,
    baseUrl = BASE_URL,
   ...
)
```

# Read next
:point_right: [Specification](/specification)
