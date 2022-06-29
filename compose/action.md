> Article level: basic.

# Actions
Check the [specification](/specification/action.md) to know more about the definition of Nimbus Actions. 

# Creating actions

```kotlin
/* 
 * Here you define the map that defines the action that will be called for each action name.
 * Example the json action below will be mapped to the showAlert function below
 
        "onPress": [{
          "_:action": "alert",
          "properties": {
            "text": "This is the alert text"
          }
        }] 
        
       
 */
val customActions: Map<String, ActionHandler> = mapOf(
        "alert" to { event: ActionEvent ->
            showAlert(event)
        }
    )

    fun showAlert(event: ActionEvent) {
        val text = event.action.properties!!["text"] as String?
        NimbusTheme.nimbusStaticState?.let {
            Toast.makeText(it.applicationContext, text, Toast.LENGTH_SHORT).show()
        }
    }
    //Then when your create the nimbus config, you can concatenate with the components map like below
    private val config = NimbusConfig(
        actions = layoutActions + customActions,
        baseUrl = BASE_URL,
        components = customComponents + layoutComponents,
    )
```

# Read next
:point_right: [Action](/action)
