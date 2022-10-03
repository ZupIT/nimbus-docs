# Operations
Check the [specification](/specification/operation.md) to know more about the definition of Nimbus Operations.

# Creating your own Operations
Nimbus already comes with some [pre-defined operations](/specification/default-operations.md), but for most applications, they're not enough and we
need to extend this set.

We'll use a function to `trim` as an example. `trim` receives a string as parameter and returns the same string but without every space before the
first character and every space after the last character.

### 1. Implement the operation
Create the function just like any other Kotlin function:

```kotlin
fun trim(value: String): String {
    return value.trim()
}
```

### 2. Register the operation
The same structure used for registering components and actions is used for registering operations: NimbusComposeUILibrary. See the example below:

```kotlin
val myAppUI = NimbusComposeUILibrary("myApp")
    .addOperation("trim") { trim(it[0] as String) },
)
```

Whenever the operation "trim" is used by a server driven view, the function `trim` will be called with its first parameter. `it` is a list containing
all arguments passed to the operation by Nimbus.

Attention: we intend to create a more intuitive and type-safe way for writing operations.

### 3. Certify your ui lib is provided in the configuration
```kotlin
private val nimbus = Nimbus(
    // ...
    ui = listOf(layoutUI, myAppUI),
)
```

# Read next
:point_right: [Actions](action.md)
