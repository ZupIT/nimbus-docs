> Article level: basic.

# Configuration

# Creating actions
Check the [action specification](/specification/action.md) to know more about the definition of Nimbus Actions. 

```kotlin
 For this json example see how to implement a custom action alert
 {
  "_:component": "material:button",
  "properties": {
    "onPress": [
      {
        "_:action": "alert",
        "properties": {
          "text": "This is the alert text"
        }
      }
    ]
  }
}
```

```kotlin
/* 
 * Here you define the map that defines the action that will be called for each action name.
 * In this example the json action "alert" will be mapped to the showAlert function below
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

# Creating operations
Check the [operation specification](/specification/operation.md) to know more about the definition of Nimbus Operations. 

```kotlin
    //Json example using the custom operation trim
    {
      "_:component": "layout:column",
      "state": {
        "id": "textWithSpaces",
        "value": "            Some text with spaces around            "
      },
      "children": [
        {
          "_:component": "layout:text",
          "properties": {
            "text": "@{textWithSpaces}"
          }
        },
        {
          "_:component": "material:button",
          "properties": {
            "text": "Trim",
            "onPress": [
              {
                "_:action": "setState",
                "properties": {
                  "path": "textWithSpaces",
                  "value": "@{trim(textWithSpaces)}"
                }
              }
            ]
          }
        }
      ]
    }
```  

```kotlin
    /* 
        Here you define that the operation with name "trim" executes the kotlin trim using the first argument
        passed to the operation and returns the trimmed string as the result
    */
     
    val trim: OperationHandler = { (it[0] as String).trim() }
    val customOperations: Map<String, OperationHandler> = mapOf(
        "trim" to trim
    )

    //Sets the custom operations on NimbusConfig
    private val config = NimbusConfig(
        operations = customOperations,
        
        // and so on...
```

# Read next
:point_right: [Specification](/specification)
