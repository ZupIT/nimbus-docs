# Disclaimer
This is a work in progress. Just like every Nimbus library, the documentation is in beta stage. Feel free to contribute!

# Introduction
The Nimbus SDUI is:

1. A solution for applications that need to have some of its user interface (UI) driven by the backend, i.e. Server Driven UI (SDUI).
1. A protocol for serializing the content and behavior of a UI into JSON so it can be sent by a backend sever and interpreted by the front-end.
1. A set of libraries that implements this protocol.

An application that uses Nimbus will have:
1. A backend that provides the JSON describing the UI.
1. A frontend application that will interpret the JSON sent by the backend and show the UI.

Nimbus libraries that are currently being implemented and are in beta stages:

- Frontend
  - [Nimbus for Jetpack Compose, Android (Kotlin)](https://github.com/ZupIT/nimbus-compose)
  - [Nimbus for SwiftUI, iOS (Swift)](https://github.com/ZupIT/nimbus-swiftui)
  - [Nimbus core (shared code between all frontend implementations - Kotlin)](https://github.com/ZupIT/nimbus-core)
- Components
  - [Layout components for Nimbus Compose](https://github.com/ZupIT/nimbus-layout-compose)
  - [Layout components for Nimbus SwiftUI](https://github.com/ZupIT/nimbus-layout-swiftui)

Nimbus libraries that are currently being implemented and are in alpha stages:
- Backend
  - [Nimbus for Node (Typescript)](https://github.com/ZupIT/nimbus-backend-ts)

Planned implementations of Nimbus (after the first stable release):
- Backend
  - Nimbus for Kotlin
- Frontend
  - Nimbus for React (web)

> Attention: the current frontend implementations for Nimbus only support Compose and SwiftUI. As of 2023, we heavily recommend their use for
developing Android and iOS applications. If, for some reason, you can't use them, you might want to take a look at
[Beagle](https://github.com/ZupIT/beagle).

These are the libs maintained by the main Nimbus team, but, as said before, Nimbus is primarily a protocol, so, as long as the JSON contract is
respected, the developers can create their own backend and frontend in their preferred languages. We encourage the community to do so and share these
new libs!

## Versioning
The versions of the Nimbus libraries are not shared. Although they will all start at 1.0.0, they will each follow their own path. The specification
(protocol) will also have its own versioning.

The versions follow the formats `a.b.c` for stable versions and `a.b.c-flag.d` for pre-releases. `a` is the major version and will be changed only
when a breaking change happens. `b` will be changed whenever a new feature is implemented. `c` will be used for small fixes.
`flag` can be either "alpha", "beta", "rc" or "snapshot" and `d` will be an incremental integer to identify the pre-release build.

## Current development stage
We're currently in beta for the frontend libs and in alpha for the backend lib.

For the next beta, the following features and fixes can be expected:
[Nimbus next features and fixes](nimbus-next.pdf)

## Getting started
- [Getting started with Nimbus for Jetpack Compose (Android)](compose/getting-started.md)
- [Getting started with Nimbus for SwiftUI (iOS)](swiftui/getting-started.md)
- [Getting started with Nimbus for Node (Backend)](backend-ts/getting-started.md)

## Documentation
- [Nimbus Specification](specification/index.md)
- [Nimbus Compose](compose/index.md)
- [Nimbus SwiftUI](swiftui/index.md)
- [Nimbus Backend TS](backend-ts/index.md)
- [Nimbus Core (shared code between the frontend libs)](core/index.md)
- [Layout components](layout/index.md)

## Samples
Nimbus has a sample application implemented for the backend, SwiftUI and Compose. Click [here](sample/index.md) to check it out.
