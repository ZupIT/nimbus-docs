# Introduction
The Nimbus SDUI is:

1. A solution for applications that need to have some of its user interface (UI) driven by the backend, i.e. Server Driven UI (SDUI).
1. A protocol for serializing the content and behavior of a UI into JSON so it can be sent by a backend sever and interpreted by the front-end.
1. A set of libraries that implements this protocol.

An application that uses Nimbus will have:
1. A backend that provides the JSON describing the UI.
1. A frontend application that will interpret the JSON sent by the backend and show the UI.

Nimbus libraries that are currently being implemented and are in beta stages:
- Backend
  - [Nimbus for Node and layout components (Typescript)](https://github.com/ZupIT/nimbus-backend-ts)
- Frontend
  - [Nimbus for Jetpack Compose, Android (Kotlin)](https://github.com/ZupIT/nimbus-compose)
  - [Nimbus for SwiftUI, iOS (Swift)](https://github.com/ZupIT/nimbus-swiftui)
- Frontend components
  - [Layout components for Nimbus Compose](https://github.com/ZupIT/nimbus-layout-compose)
  - [Layout components for Nimbus SwiftUI](https://github.com/ZupIT/nimbus-layout-swiftui)

Planned implementations of Nimbus:
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
We just released our first beta, which includes most of the features we intend to deliver in the final versions, but not necessarily in their best
possible shape. In the next months we are going to iterate over what we built so far and enhance the developer experience.

What to expect for the next betas:
- A better way for registering components. In the current version, every component must be manually deserialized from a map of properties. Our main
goal for the next beta is to find a way to automatically do this or at least make it much easier.
- Navigation: we currently only support navigating from a server driven screen to another server driven screen. We need to develop a system to allow
for a better integration between native screens and server driven screens.
- A small lib of design system components to showcase what Nimbus is capable of: button, text-input, toggle, etc.
- Hot reloading between the backend lib and the frontend libs.
- A web version of the documentation.
- Better test coverage.
- Fixes to whatever bug we find in the way.

## Getting started
- Getting started with Nimbus for Node (Backend)
- Getting started with Nimbus for Jetpack Compose (Android)
- Getting started with Nimbus for SwiftUI (iOS)

## Documentation
- Component
- Structural Components
- State
- Action
- Default Actions
- Operation
- Default Operations

## Specification
Use [this article](/specification.md) to know everything about Nimbus as an specification. this is very useful for creating your own implementation of
Nimbus.
