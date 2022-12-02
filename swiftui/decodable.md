# Decodable

The NimbusSwiftUI use the Decodable infrastructure to transform a ServerDrivenNode, ActionTriggeredEvent or [Any] into a View hierarchy, ActionDecodable or OperationDecodable respectively. This is implemented by the NimbusDecoder that do all decoding logic, such as: handle component events and children views, decoding a Any structure considering primitive types, type coercion.

For take advantage of this infrastructure you need to use the resgistration APIs that use Decodable in your signature:

```swift
public func addComponent<T: ViewDecodable>(_ name: String, _ type: T.Type) -> NimbusSwiftUILibrary

public func addAction<T: ActionDecodable>(_ name: String, _ type: T.Type) -> NimbusSwiftUILibrary

public func addOperation<T: OperationDecodable>(_ name: String, _ type: T.Type) -> NimbusSwiftUILibrary
```

## Nimbus Decodable wrappers

Using Decodable, we can get the synthesized implementation when all properties conform to the Decodable protocol, however, some properties are not Decodable by default, for example a function. To solve this problem and to ease the Decodable process, some property wrappers were created:

### @Children

This property wrapper is used to anotate a children View hierarchy in a component.

```swift
@propertyWrapper
public struct Children<Content: View> {
  public var wrappedValue: () -> Content
}
```

> Attention: When using your component with NimbusSwiftUI all children content will be an AnyView therefore you need to use AnyView type in component registration as shown in the example below.

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

### @Event and @Statefulevent

Used to anotate a event inside a Action or a Component definition. We have two different wrappers, one that wrap up a function with no parameters and another that wrap up a function with one generic parameter.

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

> Attention: All decoded events are optional and will be decoded to empty functions if the event does not exist in the json.

### @CoreAction

Used to anotate a ActionTriggeredEvent reference inside an Action.

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

This property wrapper already exists in SwiftUI to create a local mutable state within a View, however it is not decodable by default, so we implement Decodable to decode an initial value for state from a json.

```swift
extension KeyedDecodingContainer {

  public func decode<Value: Decodable>(_ type: State<Value>.Type, forKey key: Self.Key) throws -> State<Value>
  
  public func decode<Value: Decodable>(_ type: State<Value?>.Type, forKey key: Self.Key) throws -> State<Value?>
}
```

## Decodable custom

When using Decodable you can customize the deserialization by implement the `init(from: Decoder)` and declaring the `CodingKeys` by yourself. All wrappers previously explained can be used inside your `init(from: Decoder)` implementation.

In official [documentation](https://developer.apple.com/documentation/foundation/archives_and_serialization/encoding_and_decoding_custom_types) and [here](https://www.swiftbysundell.com/articles/customizing-codable-types-in-swift/) you can learn more about Decodable and customizations.

Below we have the main cases of custom serialization using NimbusSwiftUI:

### A Component with children

In this example we use the wrapper in the `init(from: decoder)` implementation to decode the Content in the content property.

```swift
struct Container<Content: View>: View, Decodable {
  var content: Content

  var body: some View {
    // body impl
  }

  public init(from: Decoder) throws {
    let container = decoder.singleValueContainer()
    content = container.decode(Children<AnyView>.self).wrappedValue()
  }
}
```

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

  public init(from: Decoder) throws {
    let container = decoder.container(keyedBy: CodingKeys.self)
    onFinish = container.decode(Event.self, forKey: .onFinish).wrappedValue
  }
}
```

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

  public init(from: Decoder) throws {
    let container = decoder.container(keyedBy: CodingKeys.self)
    content = container.decodeIfPresent(Bool.self, forKey: .bool) ?? true
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

  public init(from: Decoder) throws {
    nested = Nested(from: decoder)
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

Just declare the `CodingKeys` omitting the parameters you want to ignore.

# Read next
 :point_right: [State](state.md)