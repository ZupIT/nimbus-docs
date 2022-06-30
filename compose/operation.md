# Operations
Check the [specification](/specification/operation.md) to know more about the definition of Nimbus Operations.

# Creating your own Operations
Nimbus already comes with some [pre-defined operations](/specification/default-operations.md), but for most applications, they're not enough and we
need to extend the set of operations.

### First step
You must define the map that configures the operation name to its implementation function like below
```kotlin
val trim: OperationHandler = { (it[0] as String).trim() }
val customOperations: Map<String, OperationHandler> = mapOf(
"trim" to trim
)
```
### Second step

You pass the map of the custom operations to Nimbus
```kotlin

private val nimbus = Nimbus(
operations = customOperations,
```

# Read next
:point_right: [Actions](/action.md)
