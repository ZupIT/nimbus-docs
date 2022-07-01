> Article level: basic.

# Components
Check the [specification](/specification/component.md) to know more about the definition of Nimbus Components. 

# Creating components

When creating a new component for `NimbusSwiftUI` you need to:
- declare a SwiftUI `View`
- create a component map with type `[String: Component]`
- create and configure the `Nimbus` entrypoint

Supose that we need to construct a CustomButton with the following properties:
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

The implementation use `Button` and a `.disabled(Bool)` modifier both APIs from SwiftUI.

### Component map

To implement a component map you should conform with `Desesializable` protocol in your component:

```swift
import NimbusSwiftUI

extension CustomButton: Deserializable {
  init(from map: [String : Any]?, children: [AnyComponent]) throws {
    self.text = try getMapProperty(map: map, name: "text")
    self.enabled = try getMapPropertyDefault(map: map, name: "enabled", default: true)
    let function = getMapFunction(map: map, name: "onPress")
    self.onPress = { function(nil) }
  }
}
```

There are some utils function to help in deserialization task. To more info about usage check code documentation.

```swift
public func getMapProperty<T>(map: [String: Any]?, name: String) throws -> T

public func getMapFunction(map: [String: Any]?, name: String) -> CoreFunction

public func getMapPropertyDefault<T>(map: [String: Any]?, name: String, default: T) throws -> T

public func getMapEnumDefault<T, RawValue>(map: [String: Any]?, name: String, default: T) throws -> T where T: RawRepresentable, RawValue == T.RawValue
```

> Attention: In the function deserialization, if your function do not share the type `(Any?) -> Void` you should convert to your type wrapping the function returned from `getMapFunction(map:name:)` in another function.

Now you can create the component map:

```swift
import NimbusSwiftUI

let components: [String: Component] = [
  "material:button": { (element, children) in
    AnyComponent(try CustomButton(from: element.properties, children: children))
  }
]
```

### Nimbus entry-point
Finaly you can now configure the `Nimbus` with the component map and render the json sample:

```swift
import SwiftUI
import NimbusSwiftUI

struct ContentView: View {
  var body: some View {
    Nimbus(baseUrl: "base") {
      NimbusNavigator(json: """
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
      """)
    }
    .components(components) // component map defined in the previously step
  }
}
```

You could render if the json sample is returned from server url too:

```swift
import SwiftUI
import NimbusSwiftUI

struct ContentView: View {
  var body: some View {
    Nimbus(baseUrl: "http://server:8080") {
      NimbusNavigator(url: "/screen1.json")
    }
    .components(components)
  }
}
```

# Read next
:point_right: [Action](/action)
