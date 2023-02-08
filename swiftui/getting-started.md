# Nimbus for SwiftUI: Getting started
## 0. Pre-requisites
- [Xcode 12 or above](https://developer.apple.com/xcode/)
- iOS: 13.0 or above
- Swift 5.5 or above

## 1. Installing
You can configure the `NimbusSwiftUI` and `NimbusLayoutSwiftUI` using Xcode package or `Package.swift`:

### XCode

1. Select the menu option `File > Add Packages...`
2. Add package repository: `https://github.com/ZupIT/nimbus-layout-swiftui.git` (use "exact version" and the value 1.0.0-beta.$LATEST), where $LATEST is the latest beta build.
3. Import the package in your source files: `import NimbusLayoutSwiftUI`

### Swift Package

Add `NimbusLayoutSwiftUI` package dependency to your `Package.swift` file:

```swift
let package = Package(
  ...
  dependencies: [
    .package(url: "https://github.com/ZupIT/nimbus-layout-swiftui.git", from: "1.0.0-beta.1") // instead of beta 1, use the latest beta build.
  ],
  targets: [
    .target(name: "YourPackage",
      dependencies: [
        .product(name: "NimbusLayoutSwiftUI", package: "NimbusLayoutSwiftUI")
      ]
    ),
    ...
  ...
)
```
> Attention: We added component library `NimbusLayoutSwiftUI` and `NimbusSwiftUI` (resolved automatically). Our examples will use both, but if your project won't use the layout components, use only the `NimbusSwiftUI`(url: https://github.com/ZupIT/nimbus-swiftui.git).

## 2. Creating a Nimbus entrypoint
To start, we must create an instance of the server driven tool and pass our configuration.

```swift
struct ContentView: View {
  var body: some View {
    Nimbus(baseUrl: baseUrl) {
      // construct your view hierarchy containing a navigator here
    }
    .ui([layout])
  }
}
```

Above, `baseUrl` is the url to our backend server and `components` is the map with all components that can be rendered for a Server Driven View.
Nimbus has a lot of customizations and they're mostly done via the modifier functions for `Nimbus`. To check every possible configuration, please check the
[dedicate topic](configuration.md).

## 3. Rendering a server driven view
To render a server driven view, we need a `NimbusNavigator`. This is because, a server driven view may not be just one view, it can have instructions
like "go to another server driven view" or "go back to the previous server driven view".

A `NimbusNavigator` is a container for every server driven view that is loaded in its context. To start up a `NimbusNavigator`, we can start a with:
- url `String`
- `ViewRequest`, for request configuration
- json `String`

The example below shows the full code for rendering a server driven view hosted in Github. This will load `GET $baseUrl/hello-world.json`, i.e.
`GET https://gist.githubusercontent.com/Tiagoperes/da38171c94be043c3e5b81cbb835a0e5/raw/ab832ba276764f4de12552f02c4f56e20cd83b0b/hello-world.json`.

```swift
import SwiftUI
import NimbusLayoutSwiftUI

let BASE_URL = "https://gist.githubusercontent.com/Tiagoperes/da38171c94be043c3e5b81cbb835a0e5/raw/ab832ba276764f4de12552f02c4f56e20cd83b0b"

struct ContentView: View {
  var body: some View {
    Nimbus(baseUrl: baseUrl) {
      NimbusNavigator(url: "/hello-world.json")
    }
    .ui([layout])
  }
}
```

You should also configure your application to start with the `ContentView`:

```swift
import SwiftUI

@main
struct SampleApp: App {
  var body: some Scene {
    WindowGroup {
      ContentView()
    }
  }
}
```
## 5. Running
After running the application, you should see a "Hello World" text in the emulator.

## Read next
:point_right: [Component](/component.md)
