# Operations
Check the [specification](/specification/operation.md) to know more about the definition of Nimbus Operations.

# Creating your own Operations
Nimbus already comes with some [pre-defined operations](/specification/default-operations.md), but for most applications, they're not enough and we
need to extend this set.

We'll use a function to format a value as a currency as an example.

### 1. Implement the operation
Create the function just like any other Kotlin function:

```kotlin
import br.com.zup.nimbus.annotation.AutoDeserialize
import java.text.NumberFormat
import java.util.Currency

@AutoDeserialize
fun formatCurrency(value: Double, currency: String): String {
    val formatter: NumberFormat = NumberFormat.getCurrencyInstance()
    formatter.maximumFractionDigits = 2
    formatter.currency = Currency.getInstance(currency)
    return formatter.format(value)
}
```

### 2. Register the operation
The same structure used for registering components and actions is used for registering operations: `NimbusComposeUILibrary`. See the example below:

```kotlin
val myAppUI = NimbusComposeUILibrary("myApp")
    .addOperation("formatCurrency") { formatCurrency(it) },
)
```

> Attention: this code will be marked as invalid by the IDE until a build is performed (code generation).

Whenever the operation "formatCurrency" is used by a server driven view, the function `formatCurrency` will be called.

### 3. Register the UI Library to Nimbus
If the UI Library is not yet registered to Nimbus, please do so:

```kotlin
private val nimbus = Nimbus(
    baseUrl = BASE_URL,
    ui = listOf(layoutUI, myAppUI),
)
```

## Manual vs Automatic Action deserialization
When registering an operation, Nimbus provides all the parameters that were passed to the function call in the expression. These parameters are
wrapped in a `List` of `Any`.

Deserializing items in a list of unknown types can be very boring, repetitive and dangerous. For this reason, we recommend using the method described
in this topic (automatic), which requires both the packages "Nimbus Compose Annotations" and "Nimbus Compose Processor".

If you want to learn the manual approach, which doesn't rely on code generation, please read the topic 
["Operations: manual deserialization"](manual/action.md) instead.

# Read next
:point_right: If you want to learn how to manually deserialize an operation: [Operations: manual deserialization](manual/operation.md)
:point_right: If you want to know more about the auto-deserialization: [Auto component deserialization](auto-deserialization.md)
:point_right: Otherwise: [Actions](action.md)
