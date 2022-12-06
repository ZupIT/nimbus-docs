# Actions
Check the [specification](/specification/action.md) to know more about the definition of Nimbus Actions.

## Creating your own Actions
Nimbus already comes with some [pre-defined actions](/specification/default-actions.md), but for most applications, they're not enough and we
need to extend them.

We'll create an action that logs a message to the console as an example. This action receives two parameters:

- `message`: string, required. The message to log.
- `level`: LogLevel, optional. The type of log.
### 1. Write the Action structure

Write the structure with the parameters of the action and make it conform to the protocol `ActionDecodable`.

```swift
import NimbusSwiftUI

struct LogAction: ActionDecodable {
  enum Level: String, Decodable {
    case info
    case warning
    case error
  }

  var message: String
  var level: Level?

  func execute() {
    print("\(level ?? .info): \(message)")
  }
}
```

The `ActionDecodable` protocol must conform to `Decodable` and implement `execute()`, which is the function called when the action is triggered.
To know more about the `Decodable` usage in NimbusSwiftUI, [read this topic](decodable.md).

### 2. Register the action struct
The same structure used for registering components and operations is used for registering actions: `NimbusComposeUILibrary`. See the example below:

```swift
let myAppUI = NimbusSwiftUILibrary("myApp")
  .addAction("log", LogAction.self)
```

Whenever the action "myApp:log" is used by a server driven view, the action `log` will be executed. 

### 3. Certify your UI Library is registered to your Nimbus instance
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
```

# Read next
:point_right: If you want to learn more about the `Decodable` usage in NimbusSwiftUI: [Decodable](decodable.md).
:point_right: Otherwise: [Configuration](configuration.md)
