# Components
Check the [specification](/specification/component.md) to know more about the definition of Nimbus Components.

# Creating components

When creating a new component for `NimbusSwiftUI` you need to:
1. declare a SwiftUI `View`;
1. create a component in your `NimbusSwiftUILibrary`;
1. associate your `NimbusSwiftUILibrary` to the nimbus instance.

Suppose that we need to construct a CustomButton with the following properties:
- text: `String` showed in the button
- enabled: `Bool` indicating if the button is enabled. Default: _true_
- onPress: `() -> Void` function triggered when the button is pressed

A json sample for this component could be:
```json
{
  "_:component": "button",
  "properties": {
    "text": "Button",
    "onPress": [
      {
        "_:action": "openUrl",
        "properties": {
          "url": "https://apple.com"
        }
      }
    ]
  }
}
```

### SwiftUI View
Create your component just like any other SwiftUI `View` and add conformance with `Decodable`.

```swift
import SwiftUI

struct CustomButton: View, Decodable {
  var text: String
  var enabled: Bool?

  @Event
  var onPress: () -> Void
  
  var body: some View {
    Button(text) {
      onPress()
    }
    .disabled(!(enabled ?? true))
  }
}
```

The implementation uses `Button` and a `.disabled(Bool)` modifier, both are APIs from SwiftUI.

> Attention: You need to use `@Event` wrapper for auto Decodable conformance of the _onPress_ parameter.
To know more about the `Decodable` usage in NimbusSwiftUI, [read this topic](decodable.md).

### Component registration
The same structure used for registering actions and operations is used for registering components: `NimbusComposeUILibrary`. See the example below:

```swift
import NimbusSwiftUI

let myAppUI = NimbusSwiftUILibrary("myApp")
  .addComponent("button", CustomButton.self)
```

### Nimbus instance association
Finally, you must tell your Nimbus instance to use your UI Library:

```swift
import SwiftUI
import NimbusSwiftUI

struct ContentView: View {
  var body: some View {
    Nimbus(baseUrl: "base") {
      NimbusNavigator(json: """
      {
        "_:component": "myApp:button",
        "properties": {
          "text": "Button",
          "onPress": [
            {
              "_:action": "openUrl",
              "properties": {
                "url": "https://apple.com"
              }
            }
          ]
        }
      }
      """)
    }
    .ui([layout, myAppUI])
  }
}
```

You could also make a request instead of hardcoding the json string:

```swift
import SwiftUI
import NimbusSwiftUI

struct ContentView: View {
  var body: some View {
    Nimbus(baseUrl: "http://server:8080") {
      NimbusNavigator(url: "/screen1.json")
    }
    .ui([layout, myAppUI])
  }
}
```

# Read next

:point_right: If you want to learn more about the `Decodable` usage in NimbusSwiftUI: [Decodable](decodable.md).
:point_right: Otherwise: [State](/state.md)
