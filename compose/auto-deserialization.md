# Automatic deserialization
As seen in the documentation for [components](component.md), [actions](action.md) and [operations](operation.md), the annotation `@AutoDeserialize`
can be used for creating deserialized versions of composable functions (components), actions handlers and operations.

In order to support auto-deserialization in your project, you must make sure everything marked as "Support for auto-deserialization" in the
[getting started manual](getting-started.md#1-installing) was added to the `build.gradle`.

The automatic deserialization will check every function annotated with `@AutoDeserialize` and generate code to create another function with the
same name, at the same location that accepts `ComponentData` for components, `ActionTriggeredEvent` for action handlers and `List<Any>` for
operations. Components are identified by the annotation `@Composable`, action handlers are functions that return `Unit` and operations are functions
that returns something other than `Unit`.

This is a very powerful tool because it lets the developer concentrate in the business logic instead of dealing with deserialization.

## Supported types for deserialization
When a function is annotated with `@AutoDeserialize`, all of its parameters will be read by the annotation processor. Nimbus can deserialize every
parameter typed as:

- `String`
- `Boolean`
- `Int`
- `Long`
- `Float`
- `Double`
- `Map<String, *>`
- `List<*>`
- `Enums`
- `() -> Unit`
- `(Any?) -> Unit`
- Classes with public default constructors that accepts at least one parameter of any of these types.

Nimbus Processor generates code that will always try to make sense of the values in the property map. For instance, if the function needs a
String, but the value is an Int, the string representation of the number will be used. If the function needs an Int, but the value is a Double, the 
value is converted; and so on. To know of all the type coercion that this system makes, please check the code documentation for the class
`AnyServerDrivenData`.

In the previous list, notice that function types are accepted. These will always be interpreted as Server Driven Events, which carries [Actions]
(/specification/action.md). Events will always be properties like "onClick', "onPress", "onChange", "onFocus", "onBlur", "onSuccess", "onError", etc;
and they will run every action brought in the JSON when the function is called. The functions that accept a parameter are events that need to declare
a state value, for example, the "onChange" event of a text input declares its current value to its actions.

A class type is deserialized by the annotation processor by generating a function at the same location as the class named `deserializeClassName`,
where `ClassName` is the name of the class. This function receives the raw data and creates an instance of the class. This process is done recursively
so, if the class refers to another class, this other class will also be auto-deserialized.

Any type not mentioned in the first list can't be automatically deserialized, important mentions are:
- Sealed classes.
- Java classes.
- Type aliases.
- Abstract classes.
- Interfaces other than Map, List and Enum.
- Classes without public constructors.
- Classes with public constructors that don't accept a parameter.
- Functions that receive more than one parameter.
- Functions that return anything other than `Unit`.

## Custom deserializers
When a type can't be deserialized, we can create a custom deserialization function ourselves. It suffices to write a function that receives the data
(`AnyServerDrivenData`) and returns the desired type. See an example for `java.util.date`:

`DateDeserializer.kt`
```kt
import java.util.date
import br.com.zup.nimbus.annotation.Deserializer

@Deserializer
fun deserializeDate(data: AnyServerDrivenData): Date? = if (data.isNull()) null else Date(data.asLong())
```

Now, whenever we use the type `Date` in a component, action handler or operation annotated with `@AutoDeserialize`, instead of throwing a compilation
at build time, the Nimbus processor will use this custom deserializer.

If the type accepts a generic argument, the deserializer must be implemented for the generic argument. See an example below:

`MyAction.kt`
```kt
import br.com.zup.nimbus.annotation.AutoDeserialize
import java.util.Stack

@AutoDeserialize
fun MyAction(stack: Stack<String>)
```

`StackDeserializer.kt`
```kt
import br.com.zup.nimbus.annotation.Deserializer
import java.util.Stack

@Deserializer
fun deserializeStringStack(data: AnyServerDrivenData): Stack<String> {
    val stack = Stack<String>()
    stack.addAll(data.asList())
    return stack
}
```

A custom deserializer can also receive the [`DeserializationContext`](todo) as a parameter, it doesn't matter which comes first, the
`AnyServerDrivenData` or the `DeserializationContext`.

## Ignoring a parameter
Sometimes we may want the Nimbus Processor to completely ignore a parameter of our functions. As long as the parameter has been assigned a default
value, this can be done via the annotation `@Ignore`.

```kt
import br.com.zup.nimbus.annotation.AutoDeserialize
import br.com.zup.nimbus.annotation.Ignore

@Composable
@AutoDeserialize
fun TextInput(
    value: String,
    onChange: (String) -> Unit,
    @Ignore validate: (String) -> String? = { null },
) {
    // ...
}
```

Above, we tell the Nimbus Processor we don't care about the parameter `validate` when using the component in a server driven view.

## The Root facilitator
Sometimes, in the front-end implementation of a component or action handler, it can be hard to organize all properties at the root level. At the same 
time, creating multiple levels of properties would not be ideal for the component API (JSON, backend). To visualize this, let's use an example:

Suppose you have the components `Row` and `Column` and you need to add some styling properties to it: padding, margin, width, height and background.

```kt
import br.com.zup.nimbus.annotation.AutoDeserialize

@Composable
@AutoDeserialize
fun Row(
    // other properties
    paddingTop: Double?,
    paddingBottom: Double?,
    paddingLeft: Double?,
    paddingRight: Double?,
    padding: Double?,
    marginTop: Double?,
    marginBottom: Double?,
    marginLeft: Double?,
    marginRight: Double?,
    margin: Double?,
    width: Double?,
    height: Double?,
    background: String?,
) {
    // ...
}
```

It's great that we can specify all these optional properties at the root of the properties map of the component in the backend. But here it makes it
much harder to reuse the code in the component `Column`. Wouldn't it be much better if we received an instance of `Style` without altering the
components API? This is possible via the annotation `@Root`. See the example below:

```kt
import br.com.zup.nimbus.annotation.AutoDeserialize
import br.com.zup.nimbus.annotation.Root

class Padding(
    @Root val paddingTop: Double?,
    @Root val paddingBottom: Double?,
    @Root val paddingLeft: Double?,
    @Root val paddingRight: Double?,
    @Root val padding: Double?,
)

class Margin(
    @Root val marginTop: Double?,
    @Root val marginBottom: Double?,
    @Root val marginLeft: Double?,
    @Root val marginRight: Double?,
    @Root val margin: Double?,
)

class Size(
    @Root val width: Double?,
    @Root val height: Double?,
)

class Style(
    @Root val padding: Padding,
    @Root val margin: Margin,
    @Root val size: Size,
    val background: String?,
)

@Composable
@AutoDeserialize
fun Row(
    // other properties
    @Root style: Style,
) {
    // ...
}

@Composable
@AutoDeserialize
fun Column(
    // other properties
    @Root style: Style,
) {
    // ...
}
```

With `@Root` the code can be better organized and easily reused. This annotation informs the Nimbus Processor that it doesn't need enter a key in
the property map to instantiate the class, that all properties needed to build it is already in the current level of the map. For this reason, when
`@Root` is used on a parameter, its name makes no difference in the outcome.

When `@Root` is used in a required type, the auto-deserialization will attempt to instantiate the type every time. When `@Root` is used in an optional
type, the auto-deserialization will only attempt to instantiate the type if at least one of its properties exist in the map, otherwise it will be
assigned `null`.

It's important to notice some situations where the use of `@Root` doesn't make sense and will throw errors at build time:
- `@Root` can only be used on non-primitive types without an associated custom deserializer (`@Deserialize`).
- `@Root` can't be used in operations.
- The developer must not create cyclic references with the `@Root` annotation.

## Renaming a property
If the backend API has a component or action property with a different name than the one you'd like to use in your code, you can inform it to the
Nimbus Processor via the annotation "@Alias". See the example below:

Suppose the backend implements the action "logError" that receives the property named "error_message", but you want it to be named "message" in your
source code.

```kt
import br.com.zup.nimbus.annotation.AutoDeserialize
import br.com.zup.nimbus.annotation.Alias

@AutoDeserialize
fun logError(@Alias("error_message") message: String) {
    // ...
}
```

Although we recommend keeping the names the same between all platforms most of the times, this can be useful in some specific situations, like legacy
support.

## The DeserializationContext
This won't be necessary for most applications. But, if you need access to the `ComponentData` or `ActionTriggeredEvent` from within an auto 
deserialized component, action handler or class, you can have it by adding a parameter with type `DeserializationContext`.

When the Nimbus Processor see the type `DeserializationContext`, it injects the current context into the function. We recommend doing this as a last
resort because this will couple your application code with Nimbus.

The `DeserializationContext` contains:
- `component`: the [`ComponentData`](manual/component.md#componentdata) if this is the deserialization of a component.
- `event`: the [`ActionTriggeredEvent`](manual/action.md#actiontriggeredevent) if this is the deserialization of an action handler.

See the example below where we need to get information about the parent of the component to make a decision:

```kt
import br.com.zup.nimbus.annotation.AutoDeserialize
import br.zup.com.nimbus.compose.deserialization.DeserializationContext
import com.zup.nimbus.core.tree.ServerDrivenNode

@Composable
@AutoDeserialize
fun MyComponent(text: String, context: DeserializationContext) {
    val parent = context.component?.node?.parent
    val parentComponent = if (parent is ServerDrivenNode) parent.component else "unknown"
    Column {
        Text(text)
        Text("child of $parentComponent")
    }
}
```

This is a very specific component that needs to be aware of the component structure that came from the backend. This situation is very rare, but
can be addressed by our current solution if needed.

In action handlers this may be more common since we might want to get a reference to the component that triggered the event. See the example below:

// PAREI AQUI

## Annotations

### For functions
- `@AutoDeserialize`: makes the composable function, action handler or operation deserializable and usable as `functionName(it)` when registering it 
to a `NimbusComposeUILibrary`.
- `@Deserializer`:

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
