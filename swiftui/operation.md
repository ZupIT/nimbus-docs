# Operations
Check the [specification](/specification/operation.md) to know more about the definition of Nimbus Operations.

# Creating your own Operations
Nimbus already comes with some [pre-defined operations](/specification/default-operations.md), but for most applications, they're not enough and we
need to extend this set.

We'll use a function that validates `trim` as an example. `trim` receives a string as parameter and returns the same string but without every space before the
first character and every space after the last character.

### 1. Implement the operation
Create the function just like any other Swift function:

```swift
func trim(_ string: String?) -> String? {
  string?.trimmingCharacters(in: .whitespaces)
}
```

### 2. Register the operation
You must register the action to a UI Library.

```swift
let myAppUI = NimbusSwiftUILibrary("myApp")
  .addOperation("trim") { trim($0.get(index: 0) as? String) }
```

> Attention: right now we have to deal with untyped data and manually deserialize the list of parameters for an operation. We intend to improve this
experience before a stable release.

Whenever the operation "trim" is used by a server driven view, the function `trim` will be called with its first parameter. 

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
:point_right: [Actions](action.md)
