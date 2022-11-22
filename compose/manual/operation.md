# Operations
Check the [specification](/specification/operation.md) to know more about the definition of Nimbus Operations.

This topic will show how to manually deserialize an operation. If you want to learn the automatic and recommended approach (code generation), please
read the topic ["Operation"](../operation.md) instead.

We'll use the same component implemented [here](../operation.md#creating-your-own-operations).

You'll implement the operation just like the automatic version, but without the annotation `@AutoDeserialize`.

## Register the operation
For manual deserialization, we need to use the list of arguments passed to the operation. This list of arguments is provided as the single
parameter of the function used to register the operation. See the example below:

```kotlin
val myAppUI = NimbusComposeUILibrary("myApp")
    .addOperation("formatCurrency") { 
        val value = it[0] as Double // first argument passed to the operation
        val currency = it[1] as String // second argument passed to the operation
        return formatCurrency(value, currency)
    },
)
```

Whenever the operation "formatCurrency(x, y)" is used by an expression of a server driven view, the function `formatCurrency` will be called with a 
list where the first position corresponds to `x` and second to `y`.

The code above is just an example, it is still unsafe since we're not always checking if the value is of the type we expect before casting it. We also
have no treatment for when the expected properties are not available. To help with this, Nimbus provides the class `AnyServerDrivenData`, see below
an example of how to use it:

```kotlin
val myAppUI = NimbusComposeUILibrary("myApp")
    .addOperation("formatCurrency") {
        val data = AnyServerDrivenData(it)
        val value = data.at(0).asDouble() // first argument passed to the operation
        val currency = data.at(1).asString() // second argument passed to the operation
        if (data.hasError()) throw IllegalArgumentException(data.errorsAsString())
        return formatCurrency(value, currency)
    },
)
```

`AnyServerDrivenData` will collect the deserialization errors if they happen, without stopping the execution. It also makes automatic type coercions
for helping with type management. If you are going to use manual deserialization, we highly recommend the use of this class.

If we find any deserialization error, instead of calling the operation, we throw an exception, which will be treated and printed as an error
log by Nimbus.

# Read next
:point_right: [Actions](action.md)
