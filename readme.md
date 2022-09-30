# Disclaimer
This is a work in progress. Just like every Nimbus library, the documentation is in alpha stage. Feel free to contribute!

# Critical issues
## Components
- Components must be manually deserialized on iOS for now.
- Components can be automatically deserialized on Android most of the times, but not always. For more details check: todo.
## Custom actions
- Actions are currently untyped on both iOS and Android. Right now, a custom action handler receives a map of unknown type as the action properties 
which must manually casted to the desired type.
## Custom operations
- Operations are currently untyped on both iOS and Android. Right now, a custom operation receives a list of unknown type as its only argument. The
list represents the arguments passed to the operation in the backend and must be manually casted to the desired type.
## Navigation
- The navigation on Android is currently very buggy and causing random crashes.
- Navigation parameters are not yet implemented.
## Layout (additional library)
- We don't have yet a way of using percentage values, since this is not native to SwiftUI or Compose.

# Main differences to Beagle
- Nimbus only purpose is Server Driven UI. Additional features like components, caching, analytics or a more complex navigation system must be
provided by additional libraries or implemented by the developer. Nimbus tries to be as flexible as possible by providing extension points, but will
not implement features outside of the scope of managing Server Driven UI. Having said that, we will provide a separate library to manage layout in
SwiftUI and Compose. In the future, we also intend to provide a separate library with design system components.
- There are no stack navigations. Nimbus navigation intersects with Beagle's only at: `push`, `pop` and `popTo`. Furthermore it adds `present` and
`dismiss`, which are modal views.
- There are no ListViews. Instead, Nimbus has the core component `forEach`, which just iterates over a list and generate the UI based on a template.
The ListView behavior, if desired, must be implemented by a custom component. In most cases, however, the combination of `lazyColum/lazyRow` and
`forEach` will be enough for a great UI.
- Beagle Contexts are called States. 
- Frontend applications are unaware of expressions. Everything is handled by Nimbus, Compose and SwiftUI. The developer's only worry is to implement
the SwiftUI View or Composable function, without any coupling with Nimbus. e.g. if something in Beagle expected an expression of String, in Nimbus, it
just expects a String.

# Introduction
The Nimbus SDUI is:

1. A solution for applications that need to have some of its user interface (UI) driven by the backend, i.e. Server Driven UI (SDUI).
1. A protocol for serializing the content and behavior of a UI into JSON so it can be sent by a backend sever and interpreted by the front-end.
1. A set of libraries that implements this protocol.

An application that uses Nimbus will have:
1. A backend that provides the JSON describing the UI.
1. A frontend application that will interpret the JSON sent by the backend and show the UI.

Nimbus libraries that are currently being implemented and are in alpha stages:
- Backend
  - [Nimbus for Node (Typescript)](https://github.com/ZupIT/nimbus-backend-ts)
- Frontend
  - [Nimbus for Jetpack Compose, Android (Kotlin)](https://github.com/ZupIT/nimbus-compose)
  - [Nimbus for SwiftUI, iOS (Swift)](https://github.com/ZupIT/nimbus-swiftui)
  - [Nimbus core (shared code between all frontend implementations - Kotlin)](https://github.com/ZupIT/nimbus-core)
- Components
  - [Layout components for Nimbus Backend TS](https://github.com/ZupIT/nimbus-backend-ts/tree/main/layout)
  - [Layout components for Nimbus Compose](https://github.com/ZupIT/nimbus-layout-compose)
  - [Layout components for Nimbus SwiftUI](https://github.com/ZupIT/nimbus-layout-swiftui)

Planned implementations of Nimbus (after the first stable release):
- Backend
  - Nimbus for Kotlin backends

> Attention: the current frontend implementations for Nimbus only support Compose and SwiftUI. As of 2022, we heavily recommend their use for
developing Android and iOS applications. If, for some reason, you can't use them, you might want to take a look at
[Beagle](https://github.com/ZupIT/beagle).

These are the libs maintained by the main Nimbus team, but, as said before, Nimbus is primarily a protocol, so, as long as the JSON contract is
respected, the developers can create their own backend and frontend in their preferred languages. We encourage the community to do so and share these
new libs!

## Versioning
The versions of the Nimbus libraries are not shared. Although they will all start at 1.0.0, they will each follow their own path. The specification
(protocol) will also have its own versioning.

The versions follow the formats `a.b.c` for stable versions and `a.b.c-flag.d` for pre-releases. `a` is the major version and will be changed only
when a breaking change happens (hopefully very rarely). `b` will be changed whenever a new feature is implemented. `c` will be used for small fixes.
`flag` can be either "alpha", "beta" or "rc" and `d` will be an incremental integer to identify the pre-release build.

## Current development stage
We're currently in alpha and we expect to release the first public beta in December/2022. This is only for the frontend libs. The backend will
probably see its first beta in the beginning of 2023.

What to expect for the first beta:
- Auto deserialization for components in iOS so the developer can just link its SwiftUI Views to Nimbus instead of having to write deserializers.
- Better auto deserialization for components in Android.
- A more intuitive way for creating custom actions and custom operations without having to manually deserialize the property map.
- Navigation: bug fixes in Android and Navigation Parameters.
- Grids in the layout lib for Nimbus SwiftUI.
- Hot reloading between the backend lib and the frontend libs.

## Getting started
- [Getting started with Nimbus for Node (Backend)](backend-ts/getting-started.md)
- [Getting started with Nimbus for Jetpack Compose (Android)](compose/getting-started.md)
- [Getting started with Nimbus for SwiftUI (iOS)](swiftui/getting-started.md)

## Documentation
- [Nimbus Specification](specification/index.md)
- [Nimbus Backend TS](backend-ts/index.md)
- [Nimbus Compose](compose/index.md)
- [Nimbus SwiftUI](swiftui/index.md)
- [Nimbus Core (shared code between the frontend libs)](core/index.md)
- [Layout components](layout/index.md)

## Samples
Nimbus has a sample application implemented for the backend, SwiftUI and Compose. This sample is not yet finished, but can be a great example of
how to use the tool.

- [Description of the intended full sample](https://github.com/ZupIT/beagle-backend-ts/wiki/Sample)
- [Implementation on the backend](https://github.com/ZupIT/nimbus-backend-ts/tree/main/sample)
- [Implementation on iOS (SwiftUI)](https://github.com/ZupIT/nimbus-layout-swiftui/tree/main/Store)
- [Implementation on Android (Compose)](https://github.com/ZupIT/nimbus-layout-compose/tree/main/store)
