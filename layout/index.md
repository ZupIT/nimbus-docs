# Layout
The Nimbus specification doesn't declare any UI component, for this reason we need additional libraries to add some basic functionalities to our
Server Driven UI application. This library adds layout components to Nimbus, i.e. components that help organize other components in the screen.

# Installing

## Android
Add to your gradle file, under dependencies:
```
implementation "br.com.zup.nimbus:nimbus-layout-compose:$VERSION"
```

where `$VERSION` is the latest version.

## iOS
See ["Part 1: Installing" of the "Getting Started" for Nimbus SwiftUI](../swiftui/getting-started.md#1-Installing).

# Image Provider
This class is used to provide local images to the components "layout:localImage" and "layout:remoteImage". See the examples below of how to implement and set it up.

## Android
The ImageProvider:
```kt
class MyImageProvider(private val httpClient: HttpClient): ImageProvider {
    // rule for fetching remote images
    override suspend fun fetchRemote(url: String): Bitmap {
        val response = httpClient.sendRequest(ServerDrivenRequest(
            url = url,
            method = ServerDrivenHttpMethod.Get,
            headers = null, body = null))
        return response.let {
            BitmapFactory.decodeByteArray(
                it.bodyBytes,
                0,
                it.bodyBytes.size)
        }
    }

    // rule for fetching local images
    override fun fetchLocal(id: String): Int? = when (id) {
        "nimbus-local" -> R.drawable.nimbus_local
        "placeholder-img" -> R.drawable.placeholder_img
        else -> null
    }
}
```

Providing the ImageProvider to Nimbus:

```kt
ProvideNimbus(nimbus.imageProvider(MyImageProvider(nimbus.httpClient))) {
    // content + Nimbus Navigator
}
```

## iOS
The ImageProvider:

```swift
class MyImageProvider: ImageProvider {
  var networkClient: URLSession
  
  init(
    networkClient: URLSession = URLSession.shared
  ) {
    self.networkClient = networkClient
  }
  
  // rule for fetching remote images
  func fetch(url: String, completion: @escaping (UIImage?) -> Void) {    
    guard let url = URL(string: url) else { return }
    networkClient.dataTask(with: url) { data, response, error in
      DispatchQueue.main.async {
        guard let data = data, let image = UIImage(data: data) else {
          completion(nil)
          return
        }
        completion(image)
      }
    }.resume()
  }
  
  // rule for fetching local images
  func image(named: String) -> UIImage? {
    UIImage(named: named, in: Bundle(for: DefaultImageProvider.self), with: nil)
  }
}
```

Providing the ImageProvider to Nimbus:
```swift
Nimbus(baseUrl: "https://localhost:8080") {
  // content + NimbusNavigator
}
  .ui([layout])
  .imageProvider(MyImageProvider())
```

# Components
All components of this lib are prefixed by `layout:`. We don't have yet a full documentation for the component library, but everything you need can be found in our specification: https://gist.github.com/Tiagoperes/694516d05bf989cb0a222dd03c8f4537

In the specification, every exported type represent a component.
