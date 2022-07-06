# State
To check the definition of a State in Nimbus and its purposes, please check the [specification](specification/state).

# Local states
NimbusSwiftUI takes care of the local states for you. You don't need to worry about it! The lib processes the json and calculates the state values
before sending data to the UI layer that renders the components.

# Implicit states
It's very simple to create Components with events that use implicit states. See the example below for a component that renders a text input.

```swift
struct TextInput: View {
  @State var text: String
  var onChange: (String) -> Void
  
  var body: some View {
    TextField("Hint", text: Binding<String>(get: {
      text
    }, set: {
      text = $0
      onChange($0)
    }))
  }
}
```

Notice that the component doesn't become coupled to Nimbus. This is exactly what you would do for any Component in SwiftUI. The `onChange` function
will be provided by Nimbus and only need to be correctly deserialized in the `Deserializable` protocol implementation.

```swift
extension TextInput: Deserializable {
  init(from map: [String : Any]?, children: [AnyComponent]) throws {
    self.text = try getMapProperty(map: map, name: "text")
    
    let function = getMapFunction(map: map, name: "onChange")
    self.onChange = {
      string in function(string)
    }
  }
}
```

Now you can create the component map and configure the `Nimbus` entrypoint with it. For more information please check [Component](component.md) page.

```swift
let components: [String: Component] = [
  "textinput": { (element, children) in
    AnyComponent(try TextInput(from: element.properties, children: children))
  }
]
```

## Global State
To access the Global State in NimbusSwiftUI you must inject a core property inside you component using `Environment` property wrapper:

```swift
struct CustomComponent: View {
  @Environment(\.core) private var core: Core

  var body: some View {
    let globalState = core.globalState

    // global state logic
    
    // return your component implementation
  }
}
```

With a reference to the Global state, you can:
- read it: `globalState.getValueCopy()`. Gets a copy of the current state value.
- write to it: `globalState.set(newValue, path)`. Sets the current state value at the specified path. If no path is provided, the entire state is
replaced by `newValue`. To set user's name, for instance: `globalState.set("John", "user.name")`. If a path doesn't exist in the global context, it's
created.
- listen to changes: `onChange(listener)`. Subscribes to changes in the global state. `listener` must be a function that receives nothing and returns
nothing. The `onChange` function returns another function that, when called, removes the listener from the state.

## Read next
:point_right: [Operations](operation.md)
