> Article level: basic.

# Nimbus for Compose: Getting started
## Pre-requisites

- [Latest Android Studio (recommended)](https://developer.android.com/studio)
- [Jdk Minimum 1.8](https://www.oracle.com/java/technologies/downloads/)
- [Jetpack Compose](https://developer.android.com/jetpack/compose)
- Android API minSdk = 21 
- Kotlin 1.6.x

# Installing
Add to your application build.gradle

``` 
  // This is the nimbus compose base library
  implementation "br.com.zup.nimbus:nimbus-compose:${nimbusComposeVersion}". 
  // This is only needed if you dont want to implement the layout components yourself
  implementation "br.com.zup.nimbus:nimbus-layout-compose:${nimbusComposeLayoutVersion}" 
```

# Rendering your first server-driven screen

## Rendering with a endpoint url 


```kotlin
class MainActivity : ComponentActivity() {
    private val config = NimbusConfig(
        baseUrl = "http://myapi.com",
        components = layoutComponents,
    )

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            AppTheme {
                // A surface container using the 'background' color from the theme
                Surface(color = MaterialTheme.colors.background) {
                    Nimbus(config = config) {
                        NimbusNavigator(ViewRequest("/homepage"))
                    }
                }
            }
        }
    }
}
```
### Create on your endpoint http://myapi.com/homepage the json below
```
{
  "_:component": "layout:row",
  "children": [
    {
      "_:component": "layout:row",
      "children": [{
        "_:component": "layout:text",
        "properties": {
          "text": "r"
        }
      }],
      "properties": {
        "flex":2,
        "backgroundColor": "#FF0000"
      }
    },
    {
      "_:component": "layout:row",
      "children": [{
        "_:component": "layout:text",
        "properties": {
          "text": "g"
        }
      }],
      "properties": {
        "flex":1,
        "backgroundColor": "#00FF00"
      }
    },
    {
      "_:component": "layout:row",
      "children": [{
        "_:component": "layout:text",
        "properties": {
          "text": "b"
        }
      }],
      "properties": {
        "flex":1,
        "backgroundColor": "#0000FF"
      }
    }]
}
```

* Note you can also load this json whithin your app using the code snippet below
```
NimbusNavigator(json = YOUR_JSON)
```

### The result on your app's screen
<img src="https://github.com/ZupIT/nimbus-layout-compose/blob/main/layout/screenshots/debug/br.com.zup.nimbus.compose.layout.LayoutFlexTest_test_layout_1.png" width="228"/>


# Read next
:point_right: [Component](/components)
