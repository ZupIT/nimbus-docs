# Decodable

The NimbusSwiftUI use the Decodable infrastructure to transform a ServerDrivenNode, ActionTriggeredEvent or [Any] into View hierarchy, ActionDecodable or OperationDecodable respectively. This is implemented by the NimbusDecoder that do all Decoding needed, such as: handle component events and children views; decoding a Any structure considering primitive types; type coercion ...

For take advantage of this infrastructure you need to use the resgistration apis that use Decodable in you signature:

```swift
public func addComponent<T: ViewDecodable>(_ name: String, _ type: T.Type) -> NimbusSwiftUILibrary

public func addAction<T: ActionDecodable>(_ name: String, _ type: T.Type) -> NimbusSwiftUILibrary

public func addOperation<T: OperationDecodable>(_ name: String, _ type: T.Type) -> NimbusSwiftUILibrary
```

## Nimbus Decodable wrappers

TODO: Explicar wrappers para facilitar decoding, citar sintese automatica do protocolo

### @Children 

```swift
@propertyWrapper
public struct Children<Content: View> {
  public var wrappedValue: () -> Content
  
  public init(@ViewBuilder wrappedValue: @escaping () -> Content) {
    self.wrappedValue = wrappedValue
  }
}
```

### @Event and @Statefulevent

```swift
@propertyWrapper
public struct Event {
  public var wrappedValue: () -> Void
  
  public init(wrappedValue: @escaping () -> Void) {
    self.wrappedValue = wrappedValue
  }
}

@propertyWrapper
public struct StatefulEvent<T> {
  public var wrappedValue: (T) -> Void
  
  public init(wrappedValue: @escaping (T) -> Void) {
    self.wrappedValue = wrappedValue
  }
}
```

### @CoreAction

```swift
@propertyWrapper
public struct CoreAction {
  public var wrappedValue: ActionTriggeredEvent
  
  public init(wrappedValue: ActionTriggeredEvent) {
    self.wrappedValue = wrappedValue
  }
}
```

## Decodable custom

When using Decodable you can customize the deserialization by implement de `init(from: Decoder)` of your component by yourself.

TODO: Citar documentacao oficial

### A Component with children

```swift
struct Container<Content: View>: View {
  var content: Content

  var body: some View {
    // body impl
  }
}

extension Container: Decodable {
  public init(from: Decoder) throws {
    let container = decoder.singleValueContainer()
    content = container.decode(Children<AnyView>.self).wrappedValue()
  }
}
```

### An Action with event

```swift
struct Action: ActionDecodable {
  var onFinish: () -> Void

  func execute() {
    // action impl
  }
}

extension Action: Decodable {
  // keys

  public init(from: Decoder) throws {
    let container = decoder.singleValueContainer()
    onFinish = container.decode(Event.self).wrappedValue()
  }
}
```

### An Operation with array parameter

```swift
struct Operation: OperationDecodable {
  var array: [String]

  static var properties = ["array"]

  func execute() -> String {
    // action impl
  }
}

extension Operation: Decodable {
  // keys

  public init(from: Decoder) throws {
    let container = decoder.singleValueContainer()
    array = container.decode([String].self)
  }
}
```

### A Component with Default value

```swift
struct Default: View {
  var bool: Bool

  var body: some View {
    // body impl
  }
}

extension Default: Decodable {
  // keys

  public init(from: Decoder) throws {
    let container = decoder.singleValueContainer()
    content = container.decodeIfPresent(Bool.self) ?? true
  }
}
```

### Flatten to nested structure

```swift
struct Component: View {
  struct Nested: Decodable {
    var value: Int
  }

  var nested: Nested

  var body: some View {
    // body impl
  }
}

extension Default: Decodable {
  public init(from: Decoder) throws {
    nested = Nested(from: decoder)
  }
}
```

## Primitive registering function

TODO: Citar e explicar brevemente metodos de registro de componentes (acoes e operacoes) que nao usam Decodable.

# Read next
 :point_right: [State](state.md)