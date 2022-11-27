# Operations
Check the [specification](/specification/operation.md) to know more about the definition of Nimbus Operations.

# Creating your own Operations
Nimbus already comes with some [pre-defined operations](/specification/default-operations.md), but for most applications, they're not enough and we
need to extend them.

We'll use a function to format a value as a currency as an example.

### 1. Implement the operation
Write the structure with parameters and conforms with protocol `OperationDecodable`.

```swift
import NimbusSwiftUI

struct FormatCurrency: OperationDecodable {
  static var properties = ["value", "currency"]
  
  var value: Double
  var currency: String
  
  func execute() -> String? {
    let formatter = NumberFormatter()
    formatter.numberStyle = .currency
    formatter.currencyCode = code
    return formatter.string(from: NSNumber(value: value))
  }
}
```

The `OperationDecodable` protocol is defined as follows:

```swift
public protocol OperationDecodable: Decodable {
  associatedtype Return
  func execute() -> Return
  
  static var properties: [String] { get }
}
```

- `func execute() -> Return`: A function that holds the implementation of the operation and returns a generic type `Return`.
- `static var properties: [String]`: An array with each element being the name of the struct's properties in the order they appear in the operation.

### 2. Register the operation
The same structure used for registering components and actions is used for registering operations: `NimbusComposeUILibrary`. See the example below:

```swift
let myAppUI = NimbusSwiftUILibrary("myApp")
  .addOperation("formatCurrency", FormatCurrency.self)
```

Whenever the operation "myApp:formatCurrency" is used by a server driven view, the struct `FormatCurrency` will be executed. 

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
:point_right: Otherwise: [Actions](action.md)
