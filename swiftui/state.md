# State
To check the definition of a State in Nimbus and its purposes, please check the [specification](specification/state).

# Local states
NimbusSwiftUI takes care of the local states for you. You don't need to worry about it! The lib processes the json and calculates the state values
before sending data to the UI layer that renders the components.

# Event states
It's very simple to create Components with events that use implicit states. See the example below for a component that renders a text input.

```swift
struct TextInputTest: View, Decodable {
  var placeholder: String
  var value: String
  @StatefulEvent var onChange: (String) -> Void
  
  var body: some View {
    TextField(
      placeholder,
      text: Binding(
        get: { value },
        set: { onChange($0) }
      )
    )
  }
}
```

Your View needs to conform to the `Decodable` protocol. To take advantage of the automatic synthesis of the Decodable protocol, your function `onChange` will be provided by the `@StatefulEvent` property wrapper.

Now you can register your component and configure the `Nimbus` entrypoint with it. For more information please check [Component](component.md) page.

```swift
import SwiftUI
import NimbusSwiftUI

struct ContentView: View {
  var body: some View {
    Nimbus(baseUrl: "my-base-url") {
      // Your navigator here
    }
    .ui([layout, myAppUI])
  }
}

let myAppUI = NimbusSwiftUILibrary("myApp")
  .addComponent("textInput", TextInput.self)
```

Events always create states of their own named just like themselves. The `onChange` event above will expose a state called `onChange` for everything
in its scope.

The string passed to the `onChange` function will update the current value of the event state, making it available for anything that might need it,
like a `setState` action.

## Global State
The Global state will probably be removed in the next builds. Consider Application Managed States instead.

## Application Managed States
You can have application managed states that are shared with your server driven views.

Let's say you need a state to store all products in the shopping cart, this state must be the same throughout the entire application and it must
be accessed from both native views and Server Driven Views.

1. Create the state:
```swift
val cartState = ServerDrivenState(id: "cart", value: [])
```

2. Provide it to the nimbus instance:
```swift
struct ContentView: View {
  var body: some View {
    Nimbus(baseUrl: "base") {
       NimbusNavigator(url: "/example")
    }
    .states([cartState])
    // ...
  }
}
```

Now, the cart is accessible from within any Server Driven View of `nimbus` through the state id "cart". The current state value can be read by
calling `cartState.get()`.

Important (1): state values must be primitive types, maps of primitive types or lists of primitive types, i.e. deserializable. We do intend to make this
better before releasing a stable version.

Important (2): states don't implement the observable pattern, but, if you need, you can still observe it by attaching your logic to the Nimbus
Dependency Graph. We'll make this process far easier in the future, but for now, we recommend contacting a developer of this lib for further
information.

## Read next
:point_right: [Operations](operation.md)
