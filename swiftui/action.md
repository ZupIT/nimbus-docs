# Actions
Check the [specification](/specification/action.md) to know more about the definition of Nimbus Actions.

## Creating your own Actions
Nimbus already comes with some [pre-defined actions](/specification/default-actions.md), but for most applications, they're not enough and we
need to extend the set of actions.

We'll create an action that logs a message to the console as an example. This action receives a single parameter:

- `message`: string, required. The message to log.

### 1. Write the Action Handler
```swift
let log: Action = { event in
  if let message = event.action.properties?["message"] as? String {
    print(message)
  }  
  return nil
}
```

> Attention: right now we have to deal with untyped data and manually deserialize the property map. We intend to improve this experience before a
stable release.

We get the property "message" from the event, cast to expected type `String` and print it to the console if message is not nil.

`Action` is a typealias for `(ActionTriggeredEvent) -> Void`. An `ActionTriggeredEvent` has the following properties:

- `action`: the action triggered by the event. Use this object to find the name of the action, its properties and metadata. This will be the only
information you'll need for almost every action handler you'll write.
- `scope`: the scope where the action was triggered. This will always be a `ServerDrivenEvent` and contain information like the event name and the
entire scope hierarchy. Example of scope hierarchy: this event > parent event > node > another node > view > nimbus. Any of these scopes can be
retrieved via the `parent` pointer.
- `dependencies`: tell the Nimbus lifecycle which dependencies may have changed so it can propagate updates through its dependency graph. You'll
probably never need to use this, but it's used by the core action `setState` to update the UI after a state changes its value.

Most action handlers will only need `action.properties`.

### 2. Register the action handler
You must register the action to a UI Library.

```swift
let myAppUI = NimbusSwiftUILibrary("myApp")
  .addAction("log", handler: log)
```

Whenever the action "myApp:log" is used by a server driven view, the function `log` will be called with the event. 

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
:point_right: [Configuration](configuration.md)
