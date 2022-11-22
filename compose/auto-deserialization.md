# Automatic deserialization
As seen in the documentation for [components](component.md), [actions](action.md) and [operations](operation.md), the annotation `@AutoDeserialize`
can be used for creating deserializable versions of composable functions, actions handlers and operations.

As of alpha, we currently support as parameters of an auto-deserializable composable function:
- Primitive types
- Functions (ServerDrivenEvents)
- Enums
- Classes if the parameter is annotated with `@Root`
- Anything if the parameter is annotated with `@Ignore` or `@Computed`.

## Disclaimer
This feature is highly experimental and will probably see significant changes before release.

## Annotations

### For classes
- `@ServerDrivenComponent`: makes the composable function deserializable and usable as `Component(it)` when registering it to Nimbus.

### For function arguments
- `@Root`: forces the value to be built using the entries at the root of the property map and not the sub-map indicated by the parameter name.
- `@Ignore`: ignores the argument when deserializing. This can only be used in optional arguments.
- `@Computed(TypeDeserializer)`: uses a custom deserializer to build the argument.

## Custom deserializers (TypeDeserializer)
Sometimes you don't want to code a deserializer for the entire Composable function, but instead, just for some of its arguments. To do this, you can:

1. Build a TypeDeserializer for the argument type. See below an example for the type `AdaptiveSize` of the layout library:

```kotlin
object AdaptiveSizeDeserializer: TypeDeserializer<AdaptiveSize?> {
    override fun deserialize(
        properties: ComponentDeserializer,
        data: ComponentData,
        name: String,
    ): AdaptiveSize? {
        val sizeString = properties.asStringOrNull(name)?.lowercase()
        if (sizeString == "expand") return AdaptiveSize.Expand
        if (sizeString == "fitcontent") return AdaptiveSize.FitContent
        val sizeDouble = properties.asDoubleOrNull(name)
        return sizeDouble?.let { AdaptiveSize.Fixed(it) }
    }
}
```

Attention: the deserializer must always be an object.

2. Annotate the argument inside the Composable function. See the example below for the width and height of a Row:

```
@ServerDrivenComponent
@Composable
fun Row(
    // ...
    @Root @Computed(AdaptiveSizeDeserializer::class) val width: AdaptiveSize?
    @Root @Computed(AdaptiveSizeDeserializer::class) val height: AdaptiveSize?
) {
    // ...
}
```

We also used `@Root` because we want the `AdaptiveSize` to be deserialized using the root properties and not properties inside a sub-map called
`width` or `height`.

## Read next
:point_right: [State](state.md)
