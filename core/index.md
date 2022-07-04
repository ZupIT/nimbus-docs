# Core frontend lib
The frontend implementation of Nimbus shares a lot of code between the platforms. To achieve this we're using
[Kotlin Multiplatform Mobile (KMM)](https://kotlinlang.org/lp/mobile/).

This document lists the features that are common to both Nimbus Compose and Nimbus SwiftUI, but are also not part of the
[specification](/specification/index.md), i.e. resources that exist in this implementation of Nimbus, but are not a required by the specification.

All examples here will be given in Kotlin because this is the language Nimbus Core has been implemented with.

## Action observers
An observer that is triggered whenever an Action runs. It works similarly to the map of Actions itself, but instead of running for a specific Action,
this will run for every action.

An action observer is a function that receives an `ActionEvent` and returns nothing. An `ActionEvent` is defined as follows:

```kotlin
data class ActionEvent(
  /**
   * The action of this event. Use this object to find the name of the action, its properties and metadata.
   */
  val action: ServerDrivenAction,
  /**
   * The name of the event that triggers the action, i.e. the key of the node property that declared it. Example, a
   * component "Button" triggers the action found in the property "onPress" when it's pressed. "onPress" is the
   * "name" of this event.
   */
  val name: String,
  /**
   * The node (component) containing the action.
   */
  val node: RenderNode,
  /**
   * The view containing the node that declared the action.
   */
  val view: ServerDrivenView,
)
```

A simple use for this feature would be to log every action that is executed and has `log = true` in its metadata.

A real use case for this feature would be Analytics. We could create Analytics records based in the actions that are triggered. The data for the
analytics event could be passed in the Action's `metadata`.

Action observers are a list because more than one observer can be registered.

## Logger
The logger to call when printing errors, warning and information messages. By default, Nimbus will use its DefaultLogger that just prints the
messages to the console. This logger is also usable by the server driven views through the Action `log`.

A Logger must implement the following interface:

```kotlin
interface Logger {
  fun enable()
  fun disable()
  fun log(message: String, level: LogLevel)
  fun info(message: String)
  fun warn(message: String)
  fun error(message: String)
}
```

Where `LogLevel` is:
```kotlin
enum class LogLevel {
  Info, Warning, Error,
}
```

## URL builder
A logic to create full URLs from relative paths. By default, Nimbus checks if the path starts with "/". If it does, the full URL is the baseUrl
provided in the configuration concatenated with the path. Otherwise, it's the path itself.

A URL builder must implement the following interface:

```kotlin
interface UrlBuilder {
  /**
   * @param path the path to generate the URL (should be encoded if it contains special characters)
   * @returns the final url
   */
  fun build(path: String): String
}
```

## HTTP client
The lowest level API for making requests in Nimbus. By default, Nimbus will use its `DefaultHttpClient`, which just calls kotlin's ktor lib with the
provided requests. Replacing the Http client is required for most projects. By implementing your own, you can create authentication logic, additional
headers, retrial polices, etc.

An HTTP client must implement the following interface:

```kotlin
interface HttpClient {
  suspend fun sendRequest(request: ServerDrivenRequest): ServerDrivenResponse
}

enum class ServerDrivenHttpMethod {
  Post,
  Get,
  Put,
  Patch,
  Delete,
}

class ServerDrivenRequest(
  val url: String,
  val method: ServerDrivenHttpMethod?,
  val headers: Map<String, String>?,
  val body: String?,
)

class ServerDrivenResponse(
  val status: Int,
  val body: String,
  val headers: Map<String, String>,
  val bodyBytes: ByteArray,
)
```

## View client
The View client is one layer above the HttpClient, it is used to fetch JSON screens from the backend. By default, Nimbus use the `DefaultViewClient`,
which relies on the Http client to fetch the JSON and on the `RenderNode` object factory to deserialize the responses.

A View client is also responsible for implementing the prefetch logic, indicated by the property "prefetch" of a navigation action.

A View client must implement the following interface:

```kotlin
interface ViewClient {
  /**
   * Fetches a view (UI tree) from the server.
   *
   * @param request the data for the request to make.
   * @return the UI tree fetched.
   */
  suspend fun fetch(request: ViewRequest): RenderNode

  /**
   * Pre-fetches a view (UI tree) from the server and stores it. The next fetch will get the stored value instead of
   * performing a network request.
   *
   * Each implementation of the ViewClient decides how its pre-fetch logic works. See the class `DefaultViewClient` for the
   * default logic.
   *
   * @param request the data for the request to make.
   */
  fun preFetch(request: ViewRequest)
}
```

Where `RenderNode` is a node in the UI tree and `ViewRequest` is:

```kotlin
data class ViewRequest(
  /**
   * The URL to send the request to. When it starts with "/", it's relative to the BaseUrl.
   */
  val url: String,
  /**
   * The request method. Default is "Get".
   */
  val method: ServerDrivenHttpMethod = ServerDrivenHttpMethod.Get,
  /**
   * The headers for the request.
   */
  val headers: Map<String, String>? = null,
  /**
   * The request body. Invalid for "Get" requests.
   */
  val body: Any? = null,
  /**
   * UI tree to show if an error occurs and the view can't be fetched.
   */
  val fallback: RenderNode? = null,
)
```

## ID manager
An id generator for creating unique ids for nodes in a UI tree when one is not provided by the JSON. By default, the ids are incremental, starting at
0 and prefixed with "nimbus:".

An ID manager must implement the following interface:

```kotlin
interface IdManager {
    fun next(): String
}
```
