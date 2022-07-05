# Nimbus for Compose: Getting started
## 0. Pre-requisites
- [Latest Android Studio (recommended)](https://developer.android.com/studio)
- [Java Development Kit (JDK) 1.8 or above](https://www.oracle.com/java/technologies/downloads/)
- [Jetpack Compose](https://developer.android.com/jetpack/compose)
- Android SDK: API 21 or above
- Kotlin 1.6 or above

## 1. Installing
You can download the Nimbus library from mavenCentral by writing the following to your `build.gradle` file:

```
//Here you define that your projects need to locate dependencies on maven central repository
allprojects {
    repositories {
        mavenCentral()
    }
}
```

``` 
  // Nimbus compose base library
  implementation "br.com.zup.nimbus:nimbus-compose:${nimbusComposeVersion}". 
  // Recommended if you don't want to implement the layout components yourself
  implementation "br.com.zup.nimbus:nimbus-layout-compose:${nimbusComposeLayoutVersion}" 
```
Above, we added both the core Nimbus library and a component library for layout components. Our examples will use both, but if your project won't use the layout components, you don't need to the second dependency.

## 2. Creating an instance of Nimbus
To start, we must create an instance of the server driven tool and pass our configuration.

```kotlin
const val BASE_URL = "https://gist.githubusercontent.com/hernandazevedozup/eba4f2eb6afd6d6769a549fe037c1613/raw/cd3a897f4384783a1e799bb118a0dbfa8838fcf0"

class MainActivity : ComponentActivity() {
    private val nimbus = Nimbus(
        baseUrl = BASE_URL,
        components = layoutComponents,
    )
}
```

Above, `baseUrl` is the url to our backend server and `components` is the map with all components that can be rendered for a Server Driven View.
Nimbus has a lot of customizations and they're mostly done via the constructor for `Nimbus`. To check every possible configuration, please check the
[dedicate topic](configuration.md).

Above, we used a gist in Github to show an example. In real applications, this would be the url to your backend server, i.e. the application that
provides the JSON for the server driven views.

## 3. Providing nimbus to the UI tree
The next step is to tell the UI tree that, from that node and forward, every time nimbus is requested, the instance of nimbus we just created must
be used.

By allowing the UI tree to dictate which nimbus instance to use, we can use different instances of nimbus depending on where in the UI tree we are.
Although this is not important for most projects, this is essencial for projects with multiple modules.

To provide nimbus, we must use the component `ProvideNimbus`, that accepts a single argument: the nimbus instance.

```kotlin
class MainActivity : ComponentActivity() {
    // create the nimbus instance (previous example)

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            AppTheme {
                // A surface container using the 'background' color from the theme
                Surface(color = MaterialTheme.colors.background) {
                    // Providing the nimbus configuration to the render tree
                    ProvideNimbus(nimbus) {
                        // your content goes here. Every node from here and on will use the provided nimbus instance
                    }
                }
            }
        }
    }
}
```

## 4. Rendering a server driven view
To render a server driven view, we need a Nimbus Navigator. This is because, a server driven view may not be just one view, it can have instructions
like "go to another server driven view" or "go back to the previous server driven view".

A Nimbus Navigator is a container for every server driven view that is loaded in its context. To start up a Nimbus Navigator, we need to tell which
server driven view is the first of its navigation stack. This is done via the single argument it accepts: a `ViewRequest`.

The example below shows the full code for rendering a server driven view hosted in Github. This will load `GET $baseUrl/1`, i.e.
`GET https://gist.githubusercontent.com/hernandazevedozup/eba4f2eb6afd6d6769a549fe037c1613/raw/cd3a897f4384783a1e799bb118a0dbfa8838fcf0/1`.

```kotlin
const val BASE_URL = "https://gist.githubusercontent.com/hernandazevedozup/eba4f2eb6afd6d6769a549fe037c1613/raw/cd3a897f4384783a1e799bb118a0dbfa8838fcf0"

class MainActivity : ComponentActivity() {
    private val nimbus = Nimbus(
        baseUrl = BASE_URL,
        components = layoutComponents,
    )

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            AppTheme {
                Surface(color = MaterialTheme.colors.background) {
                    ProvideNimbus(nimbus) {
                        NimbusNavigator(ViewRequest("/1"))
                    }
                }
            }
        }
    }
}
```

## 5. Running
After running the application, you should see the following interface in the emulator's screen:
<img src="https://github.com/ZupIT/nimbus-layout-compose/blob/main/layout/screenshots/debug/br.com.zup.nimbus.compose.layout.LayoutFlexTest_test_layout_1.png" width="228"/>

## Read next
:point_right: [Component](/components)
