# Decodable

Nimbus, in its core, doesn't know what the properties of a component or action are, these are represented by Dictionaries of unknown type. The same can be said for operations, Nimbus doesn't know how many arguments or what type of arguments are accepted, internally, it's just an array of unknown type.

In order to render components, execute actions and retrieve the result of operations, Nimbus must transform these unknown types into the types accepted by the `Views`, `ActionDecodables` and `OperationDecodables` registered to the `NimbusSwiftUILibrary`. We call this process "deserialization" and we tried to make it as seamless as possible to the developer.

Nimbus uses the feature `Decodable` from Swift to make the deserialization automatic. Under the hood, the `NimbusDecoder` is used to deserialize the unknown types into the types expected by the struct. This conversion process includes error handling and type coercion, i.e. if the property or argument is of unexpected type, the decoder will first try to make sense of it, if it can't, it will log an error and not render the component or run the action or operation.

`NimbusDecoder` deserializes the actual types into the expected types according to the following rules:
- If the struct requires a string, the decoder will accept anything, but null. It uses the string representation of the type if it's not a string.
- If the struct requires a number type, the decoder will accept any type of number, truncating the original value if the expected type can't hold it. It also accepts any string that is in the format of a number, examples: `"725487"`,  `"537.5975"`.
- If the struct requires a boolean, the decoder will only accept boolean.
- If the struct requires an enum, the decoder will only accept a string that represents some of the possibilities. This matching is case-insensitive.
- If the struct requires a function, the decoder will only accept a `ServerDrivenEvent` and will deserialize it into a call to the method `run` of this structure.
- If the struct requires a non-primitive type, this type must de decodable.

To use the `NimbusDecoder` to automatic deserialize your components, actions and operations, register them using the methods that accept a Decodable:

```swift
public func addComponent<T: ViewDecodable>(_ name: String, _ type: T.Type) -> NimbusSwiftUILibrary

public func addAction<T: ActionDecodable>(_ name: String, _ type: T.Type) -> NimbusSwiftUILibrary

public func addOperation<T: OperationDecodable>(_ name: String, _ type: T.Type) -> NimbusSwiftUILibrary
```

## Nimbus Decodable wrappers

Using Decodable, we can get the synthesized implementation when all properties conform to the Decodable protocol, however, some properties are not Decodable by default, for example, a function. To solve this problem and to ease the Decodable process, some property wrappers were created:

### @Children

This property wrapper is used to annotate the child View hierarchy in a component.

```swift
@propertyWrapper
public struct Children<Content: View> {
  public var wrappedValue: () -> Content
}
```

> Attention: When using your component with NimbusSwiftUI all child content will be of the type `AnyView`, therefore you need to use an `AnyView` in the component registration as shown in the example below.

```swift
struct Container<Content: View>: View, Decodable {
  @Children var content: () -> Content

  var body: some View {
    // body impl
  }
}

let myAppUI = NimbusSwiftUILibrary("myApp")
  .addComponent("container", Container<AnyView>.self)
```

### @Event and @StatefulEvent

Used to annotate an event inside an Action or a Component definition. We have two different wrappers, one that wraps up a function with no parameters and another that wraps up a function with one generic parameter.

```swift
@propertyWrapper
public struct Event {
  public var wrappedValue: () -> Void
}

@propertyWrapper
public struct StatefulEvent<T> {
  public var wrappedValue: (T) -> Void
}
```

> Attention: All decoded events are optional and will be decoded to empty functions if the event does not exist in the source property map (dictionary) or argument list (array).

### @CoreAction

Used to annotate an `ActionTriggeredEvent` reference inside an Action.

```swift
@propertyWrapper
public struct CoreAction {
  public var wrappedValue: ActionTriggeredEvent
}
```

This reference can be used to access some core dependencies inside an Action, as shown in the example below.

```swift
import NimbusCore

struct Action: ActionDecodable {
  @CoreAction var actionObject: ActionTriggeredEvent

  func execute() {
    actionObject.scope.nimbus.logger.info(message: "log message!")
  }
}
```

### SwiftUI @State

This property wrapper already exists in SwiftUI to create a local mutable state within a View, however it is not decodable by default, so we implement `Decodable` to decode an initial value for the state from the source property map (dictionary) or argument list (array).

```swift
extension KeyedDecodingContainer {

  public func decode<Value: Decodable>(_ type: State<Value>.Type, forKey key: Self.Key) throws -> State<Value>
  
  public func decode<Value: Decodable>(_ type: State<Value?>.Type, forKey key: Self.Key) throws -> State<Value?>
}
```

## Custom Decodable

When using Decodable you can customize the deserialization by implementing `init(from: Decoder)` and declaring the `CodingKeys` by yourself. All wrappers previously explained can be used inside your `init(from: Decoder)` implementation.

In the official [documentation](https://developer.apple.com/documentation/foundation/archives_and_serialization/encoding_and_decoding_custom_types) and [here](https://www.swiftbysundell.com/articles/customizing-codable-types-in-swift/) you can learn more about Decodable and customizations.

Below we have the main cases of custom serialization using NimbusSwiftUI:

### A Component with children

In this example we use the wrapper in the `init(from: decoder)` implementation to decode the Content in the content property.

```swift
struct Container<Content: View>: View, Decodable {
  var content: () -> Content

  var body: some View {
    // body impl
  }

  init(from decoder: Decoder) throws {
    let container = try decoder.singleValueContainer()
    content = try container.decode(Children<Content>.self).wrappedValue
  }
}
```

This is equivalent to use the property wrapper @Children.

### An Action with event

In this example we use the wrapper in the `init(from: decoder)` implementation to decode the onFinish function.

```swift
struct Action: ActionDecodable {
  var onFinish: () -> Void

  func execute() {
    // action impl
  }

  enum CodingKeys: CodingKey {
    case onFinish
  }

  init(from decoder: Decoder) throws {
    let container = try decoder.container(keyedBy: CodingKeys.self)
    onFinish = try container.decode(Event.self, forKey: .onFinish).wrappedValue
  }
}
```

This is equivalent to use the property wrapper @Event.

### A Component with Default value

In this example we have a parameter bool with default value `true`.

```swift
struct Component: View, Decodable {
  var bool: Bool = true

  var body: some View {
    // body impl
  }

  enum CodingKeys: CodingKey {
    case bool
  }

  init(from decoder: Decoder) throws {
    let container = try decoder.container(keyedBy: CodingKeys.self)
    content = try container.decodeIfPresent(Bool.self, forKey: .bool) ?? true
  }
}
```

### Flatten to nested structure

This example show how to decode a flat structure where the `value` parameter comes at the root of the `properties` map and we want to transform it into a nested object where the `value` goes to the `Nested` object.

```swift
struct Component: View, Decodable {
  struct Nested: Decodable {
    var value: Int
  }

  var nested: Nested

  var body: some View {
    // body impl
  }

  init(from decoder: Decoder) throws {
    nested = try Nested(from: decoder)
  }
}
```

For the nested struture you need to instantiate it using the `init(from: Decoder)` initializer.

### Ignoring a property

This example shows how to ignore some properties in decoding.

```swift
struct Component: View, Decodable {
  var int: Int
  var ignoredInt: Int?
  @State var ignoredState: Int = 0

  var body: some View {
    // body impl
  }

  enum CodingKeys: CodingKey {
    case int
  }
}
```

Just declare the `CodingKeys` omitting the parameters you want to ignore. This is very useful for declaring internal states that shouldn't interact with the properties of a component.

# Read next
 :point_right: [Configuration](configuration.md)