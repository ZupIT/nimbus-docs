# Operations
Check the [specification](/specification/operation.md) to know more about the definition of Nimbus Operations.

# Creating your own Operations
Nimbus already comes with some [pre-defined operations](/specification/default-operations.md), but for most applications, they're not enough and we
need to extend this set.

We'll use a function that validates `trim` as an example. `trim` receives a string as parameter and returns the same string but without every space before the
first character and every space after the last character.

### 1. Implement the operation
Create the function just like any other Kotlin function:

```kotlin
fun trim(value: String): String {
    return value.trim()
}
```

### 2. Register the operation
You must create the structure that maps the operation name to its implementation. This is the map we pass in the parameter `operations` when creating
an instance of Nimbus. See the example below:

```kotlin
val customOperations: Map<String, OperationHandler> = mapOf(
    "trim" to { trim(it[0] as String) }
)
```

Whenever the operation "trim" is used by a server driven view, the function `trim` will be called with its first parameter. 

### 3. Certify your operation map is provided in the configuration
```kotlin
private val nimbus = Nimbus(
    baseUrl = BASE_URL,
    operations = customOperations,
)
```

# Read next
:point_right: [Actions](action.md)
