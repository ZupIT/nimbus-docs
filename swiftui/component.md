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
Implementing the custom component:

```swift
import SwiftUI

struct CustomButton: View {
  var text: String
  var enabled: Bool = true
  var onPress: () -> Void
  
  var body: some View {
    Button(text) {
      onPress()
    }
    .disabled(!enabled)
  }
}
```

The implementation uses `Button` and a `.disabled(Bool)` modifier, both are APIs from SwiftUI.

### Deserialization

To allow Nimbus to use your component, you need to tell it how to deserialize a property map into the parameters expected by the View. To do
this, you should make your View conform to the `Deserializable` protocol:

```swift
import NimbusSwiftUI

extension CustomButton: Deserializable {
  init(from map: [String : Any]?, children: [AnyComponent]) throws {
    self.text = try getMapProperty(map: map, name: "text")
    self.enabled = try getMapPropertyDefault(map: map, name: "enabled", default: true)
    let onPressEvent = getMapEvent(map: map, name: "onPress")
    self.onPress = { onPressEvent.run() }
  }
}
```

There are some utils function to help in deserialization task. To more info about usage check code documentation.

```swift
public func getMapProperty<T>(map: [String: Any]?, name: String) throws -> T

public func getMapEvent(map: [String: Any]?, name: String) -> ServerDrivenEvent

public func getMapPropertyDefault<T>(map: [String: Any]?, name: String, default: T) throws -> T

public func getMapEnumDefault<T, RawValue>(map: [String: Any]?, name: String, default: T) throws -> T where T: RawRepresentable, RawValue == T.RawValue
```

> Attention: the component deserialization should be a more automatic process and we intend to improve it a lot before release.

### Component registration
You must register the component to a UI Library:

```swift
import NimbusSwiftUI

let myAppUI = NimbusSwiftUILibrary("myApp")
  .addComponent("button") { (element, children) in
    AnyComponent(try CustomButton(from: element.properties, children: children))
  }
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
:point_right: [State](/state.md)
