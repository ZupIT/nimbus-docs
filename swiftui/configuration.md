# Nimbus entrypoint
When creating the `Nimbus` entrypoint we only need the `baseUrl`, the remaining configurable properties will be done via function modifiers. In this topic we'll see every
configurable property, what they do and examples of how to change the default behavior.

## baseUrl
The base url to use when dealing with relative URLs in both the frontend and the server driven views (JSONs). This should be the URL to the server
that provides the server driven views.

```swift
Nimbus(baseUrl: BASE_URL) {
  // hierarchy containing a navigator
}
```

## ui
The list of libraries containing the [components](component.md), [actions](action.md) and [operations](operation.md) for building the server driven
views.

```swift
extension Nimbus {
  func ui(_ ui: [NimbusSwiftUILibrary]) -> Nimbus
}
```

## logger
The application's [Logger](/core/index.md#logger).

```swift
extension Nimbus {
  func logger(_ logger: Logger) -> Nimbus
}
```

Example of Logger: TODO.

## urlBuilder
The application's [Url builder](/core/index.md#url-builder).

```swift
extension Nimbus {
  func urlBuilder(_ urlBuilder: (String) -> UrlBuilder) -> Nimbus
}
```

Example of URL Builder: TODO.

## httpClient
The application's [HTTP client](/core/index.md#http-client).

```swift
extension Nimbus {
  func httpClient(_ httpClient: HttpClient) -> Nimbus
}
```

Example of HTTP client: TODO.

## viewClient
The application's [View client](/core/index.md#view-client).

```swift
extension Nimbus {
  func viewClient(_ viewClient: (Core) -> ViewClient) -> Nimbus // Core contains every nimbus dependency
}
```

Example of View client: TODO.

## idManager
The application's [ID manager](/core/index.md#id-manager).

```swift
extension Nimbus {
  func idManager(_ idManager: IdManager) -> Nimbus
}
```

Example of ID manager: TODO.

## loadingView
The SwiftUI view to render when a server driven view is loading. By default, it renders a spinner (`ActivityIndicator`).

```swift
extension Nimbus {
  func loading<LoadingView: View>(@ViewBuilder loading: @escaping () -> LoadingView) -> Nimbus
}
```

Example of a custom loading component: TODO.

## errorView
The SwiftUI view to render when a server driven view fails to load and no fallback was provided. The error view always receive 2 arguments:
1. the error itself (`Error`);
2. a retry function (`() -> Void`) that when called sends the request again, replacing the error component with the loading component.

By default, it renders a simple text message with a button to retry.

```swift
extension Nimbus {
  func error<ErrorView: View>(@ViewBuilder error: @escaping (Error, @escaping () -> Void) -> ErrorView) -> Nimbus
}
```

Example of a custom error component: TODO.
