# Nimbus for Compose: Getting started
## 0. Pre-requisites
- [Latest Android Studio (recommended)](https://developer.android.com/studio)
- [Java Development Kit (JDK) 1.8 or above](https://www.oracle.com/java/technologies/downloads/)
- [Jetpack Compose](https://developer.android.com/jetpack/compose)
- Android SDK: API 21 or above
- Kotlin 1.6 or above

## 1. Installing
You can download the Nimbus library from mavenCentral by writing the following to the `build.gradle` files:

`root build.gradle`
```kt
// Here you define that your projects need to locate dependencies on maven central repository
allprojects {
    repositories {
        mavenCentral()
    }
}
```

`app build.gradle`
```kt
plugins {
    // Support for auto-deserialization
    id("com.google.devtools.ksp") version "1.7.10-1.0.6" // use the latest version of KSP according to your Kotlin version
}

dependencies {
    // Nimbus compose base library
    implementation("br.com.zup.nimbus:nimbus-compose:$version")
    // Layout components for Nimbus
    implementation("br.com.zup.nimbus:nimbus-layout-compose:$version")
    // Support for auto-deserialization
    implementation("br.com.zup.nimbus:nimbus-compose-annotation:$version")
    ksp("br.com.zup.nimbus:nimbus-compose-processor:$version")
}

// Support for auto-deserialization
kotlin {
    sourceSets.debug {
        kotlin.srcDir("build/generated/ksp/debug/kotlin")
    }
    sourceSets.release {
        kotlin.srcDir("build/generated/ksp/release/kotlin")
    }
    sourceSets.test {}
}
```

You should use the [most recent version of Nimbus Compose](https://mvnrepository.com/artifact/br.com.zup.nimbus/nimbus-compose) for `$version`.

We added both the core Nimbus library and a component library for layout components. Our examples will use both, but if your project won't use
the layout components, you don't need to the second dependency.

We also added KSP, Nimbus Annotations and Nimbus Processor for generating code for the auto-deserialization of components, action handlers and 
operations. If you're going to use manual deserialization only (not recommended), you can remove these dependencies.

## 2. Adding permissions
Just like any other Android application that makes network requests. To use Nimbus, you must add Internet permissions to your manifest. For more details
on this, please check [the official Android documentation](https://developer.android.com/training/basics/network-ops/connecting).

## 3. Creating an instance of Nimbus
To start, we must create an instance of the server driven tool and pass our configuration.

```kotlin
const val BASE_URL = "https://gist.githubusercontent.com/Tiagoperes/da38171c94be043c3e5b81cbb835a0e5/raw/ab832ba276764f4de12552f02c4f56e20cd83b0b"

class MainActivity : ComponentActivity() {
    private val nimbus = Nimbus(
        baseUrl = BASE_URL,
        ui = listOf(layoutUI),
        mode = if (BuildConfig.DEBUG) NimbusMode.Development else NimbusMode.Release,
    )
}
```

Above, `baseUrl` is the url to our backend server and `ui` is a list of `ComposeUILibrary` with all components, actions and operations that can be
rendered for a Server Driven View.
Nimbus has a lot of customizations and they're mostly done via the constructor for `Nimbus`. To check every possible configuration, please check the
[dedicate topic](configuration.md).

Above, we used a gist in Github to show an example. In real applications, this would be the url to your backend server, i.e. the application that
provides the JSON for the server driven views.

## 4. Providing nimbus to the UI tree
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

## 5. Rendering a server driven view
To render a server driven view, we need a Nimbus Navigator. This is because, a server driven view may not be just one view, it can have instructions
like "go to another server driven view" or "go back to the previous server driven view".

A Nimbus Navigator is a container for every server driven view that is loaded in its context. To start up a Nimbus Navigator, we need to tell which
server driven view is the first of its navigation stack. This is done via the single argument it accepts: a `ViewRequest` or a JSON string.

The example below shows the full code for rendering a server driven view hosted in Github. This will load `GET $baseUrl/hello-world.json`, i.e.
`GET https://gist.githubusercontent.com/Tiagoperes/da38171c94be043c3e5b81cbb835a0e5/raw/ab832ba276764f4de12552f02c4f56e20cd83b0b/hello-world.json`.

```kotlin
const val BASE_URL = "https://gist.githubusercontent.com/Tiagoperes/da38171c94be043c3e5b81cbb835a0e5/raw/ab832ba276764f4de12552f02c4f56e20cd83b0b"

class MainActivity : ComponentActivity() {
    private val nimbus = Nimbus(
        baseUrl = BASE_URL,
        ui = listOf(layoutUI),
    )

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            AppTheme {
                Surface(color = MaterialTheme.colors.background) {
                    ProvideNimbus(nimbus) {
                        NimbusNavigator(ViewRequest("/hello-world.json"))
                    }
                }
            }
        }
    }
}
```

## 6. Running
After running the application, you should see a "Hello World" text in the emulator.

## Read next
:point_right: [Component](component.md)
