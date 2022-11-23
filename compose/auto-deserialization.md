# Automatic deserialization
As seen in the documentation for [components](component.md), [actions](action.md) and [operations](operation.md), the annotation `@AutoDeserialize`
can be used for creating deserialized versions of composable functions (components), action handlers and operations.

In order to support auto-deserialization in your project, you must make sure everything marked as "Support for auto-deserialization" in the topic
["getting started"](getting-started.md#1-installing) was added to the `build.gradle`.

The automatic deserialization will check every function annotated with `@AutoDeserialize` and generate code to create another function with the
same name, at the same location that accepts `ComponentData` for components, `ActionTriggeredEvent` for action handlers and `List<Any>` for
operations. Components are identified by the annotation `@Composable`, action handlers are functions that return `Unit` and operations are functions
that returns something other than `Unit`.

This is a very powerful tool because it lets the developer concentrate in the business logic instead of dealing with deserialization.

## Summary
- [`@AutoDeserialize`](#automatic-deserialization): can be used on components (composable functions), action handlers and operations. It tells the 
Nimbus Processor to create a version of the function that accepts the raw data from the backend (JSON) and calls the original function with the 
desired types.
- [`@Deserialize`](#custom-deserializers): can be used on functions to create a custom deserializer for types that are not supported by the Nimbus 
Processor or types that you want a different strategy than the default.
- [`@Ignore`](#ignoring-a-parameter): can be used on parameters in order to inform Nimbus Processor to ignore them.
- [`@Root`](#the-root-facilitator): can be used on parameters in order to inform Nimbus Processor not to consider them as keys and continue 
to deserialize the same level of the map.
- [`@Alias`](#renaming-a-property): can be used on parameters in order to inform Nimbus Processor that they have another name in the backend.
- [`DeserializationContext`](#the-deserializationcontext): this type can be used on a parameter to tell Nimbus Processor to inject the current
deserialization context.

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
- `Enum<*>`
- `() -> Unit`: interpreted as `ServerDrivenEvent`.
- `(Any?) -> Unit`: interpreted as `ServerDrivenEvent` with state.
- `@Composable (*) -> Unit`: interpreted as the children (content) of the component. `*` here means any number of parameters, of any type.
- Classes with public default constructors that accepts at least one parameter of any of these types.

Nimbus Processor generates code that will always try to make sense of the values in the property map. For instance, if the function needs a
String, but the value is an Int, the string representation of the number will be used. If the function needs an Int, but the value is a Double, the 
value is converted; and so on. To know all about the type coercion this system makes, please check the code documentation for the class
[`AnyServerDrivenData`](https://github.com/ZupIT/nimbus-core/blob/main/src/commonMain/kotlin/com/zup/nimbus/core/deserialization/AnyServerDrivenData.kt).

In the previous list, notice that function types are accepted. These will always be interpreted as Server Driven Events when not annotated with
`@Composable`. Server Driven Events carry [Actions](/specification/action.md) and will always be properties like "onClick', "onPress", 
"onChange", "onFocus", "onBlur", "onSuccess", "onError", etc. Once called, the function will run every action brought in the JSON.

The functions that accept a parameter are events that need to declare a state value, for example, the "onChange" event of a text input declares its 
current value in the state "onChange".

Parameters that are functions annotated with `@Composable` will receive the composable function that renders the children of the component. If the
component has no children, they render an empty `Column`.

> Attention: the composable function type is only accepted as a parameter of a composable function (component). An error at build time will be raised
if this type is used for an action handler or operation. The other types of functions (events), are not acceptable inside operations.

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
- Functions that receive more than one parameter and are not composable.
- Functions that return anything other than `Unit`.

## Custom deserializers
When a type can't be deserialized, we can create a deserialization function ourselves. It suffices to write a function that receives the data
(`AnyServerDrivenData`) and returns the desired type. See an example for `java.util.Date`:

`DateDeserializer.kt`
```kt
import java.util.date
import br.com.zup.nimbus.annotation.Deserializer

@Deserializer
fun deserializeDate(data: AnyServerDrivenData): Date? = if (data.isNull()) null else Date(data.asLong())
```

Now, whenever we use the type `Date?` in a component, action handler or operation annotated with `@AutoDeserialize`, instead of throwing a compilation
error at build time, the Nimbus processor will use this custom deserializer.

If the type accepts a generic argument, the deserializer must be implemented for the generic argument. See an example below:

`MyAction.kt`
```kt
import br.com.zup.nimbus.annotation.AutoDeserialize
import java.util.Stack

@AutoDeserialize
fun MyAction(stack: Stack<String>) {
    // ...
}
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

A custom deserializer can also receive the [`DeserializationContext`](#the-deserializationcontext) as a parameter, it doesn't matter which comes 
first, the `AnyServerDrivenData` or the `DeserializationContext`.

## Default parameter values
We can't read a default parameter value from Kotlin Symbol Processor (KSP). For this reason, be aware that whenever you use a default value for a
parameter, this value will apply only for your own calls and not for server driven views. See the example below:

```kt
@Composable
@AutoDeserialize
fun Button(label: String, onPress: () -> Unit, enabled: Boolean = true) {
    Button(onClick = onPress, enabled = enabled) {
        Text(label)
    }
}
```

If the JSON doesn't specify any value for the property `enabled` or specifies null, the deserialization will fail, because it expects a boolean value
to be available, remember: default values can't be seen by the annotation processor.

For this reason, we advise the developer to not use default values and instead, accept optional parameters:

```kt
@Composable
@AutoDeserialize
fun Button(label: String, onPress: () -> Unit, enabled: Boolean?) {
    Button(onClick = onPress, enabled = enabled != false) {
        Text(label)
    }
}
```

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

It's great that we can specify all these optional properties at the root of the properties map in the backend. But here it makes it much harder to 
reuse the code in the component `Column`. Wouldn't it be much better if we received an instance of `Style` without altering the components API? This 
is possible via the annotation `@Root`. See the example below:

```kt
import br.com.zup.nimbus.annotation.AutoDeserialize
import br.com.zup.nimbus.annotation.Root

class Padding(
    val paddingTop: Double?,
    val paddingBottom: Double?,
    val paddingLeft: Double?,
    val paddingRight: Double?,
    val padding: Double?,
)

class Margin(
    val marginTop: Double?,
    val marginBottom: Double?,
    val marginLeft: Double?,
    val marginRight: Double?,
    val margin: Double?,
)

class Size(
    val width: Double?,
    val height: Double?,
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

When `@Root` is used on a required type, the auto-deserialization will attempt to instantiate the type every time. When `@Root` is used on an optional
type, the auto-deserialization will only attempt to instantiate the type if at least one of its properties exist in the map, otherwise it will be
assigned `null`.

`@Root` can't be used in every scenario, here are some cases where the use of `@Root` doesn't make sense and will throw errors at build time:
- `@Root` can only be used on non-primitive types without an associated custom deserializer (`@Deserialize`).
- `@Root` can't be used in operations.
- The developer must not create cyclic references with the `@Root` annotation.

## Renaming a property
If the backend API has a component or action property with a different name than the one you'd like to use in your code, you can inform it to the
Nimbus Processor via the annotation `@Alias`. See the example below:

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

Although we recommend keeping the names the same between all platforms, this can be useful in some specific situations, like legacy support.

## The DeserializationContext
This won't be necessary for most applications. But, if you need access to the `ComponentData` or `ActionTriggeredEvent` from within an auto 
deserialized component, action handler or class, you can have it by adding a parameter with type `DeserializationContext`.

When Nimbus Processor see the type `DeserializationContext`, it injects the current context into the function. We recommend doing this as a last
resort because this will couple your application code with Nimbus.

The `DeserializationContext` contains:
- `component`: the [`ComponentData`](manual/component.md#componentdata) if this is the deserialization of a component.
- `event`: the [`ActionTriggeredEvent`](manual/action.md#actiontriggeredevent) if this is the deserialization of an action handler.

See the example below where we need to get information about the parent of the component:

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

In action handlers this may be more common since we might want to get a reference to some of the services registered to the current instance of
Nimbus. See the example below where we use the Logger to write a log message.

```kt
import br.com.zup.nimbus.annotation.AutoDeserialize
import br.zup.com.nimbus.compose.deserialization.DeserializationContext

@AutoDeserialize
fun log(message: String, context: DeserializationContext) {
    val logger = context.event?.scope?.nimbus?.logger
    logger?.error?.let { it(message) }
}
```

# Read next
:point_right: [State](state.md)
