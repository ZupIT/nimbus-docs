# Navigation actions
These actions are responsible for performing server driven navigation within Nimbus, i.e. they're responsible for replacing the current server driven
screen with another server driven screen, using the native navigation of the platform at hand.

There are two types of navigation in Nimbus:
- Common: performs a navigation in full screen, replaces the full content of the screen, the previous screen is not visible after the navigation
completes.
- Presentation: shows the new screen as a dialog, the previous screen is still visible underneath.

# Common navigation actions
## push
This action is responsible for pushing a new screen to the navigation stack. The new screen becomes visible and the old screen is hidden. A platform
specific transition might be played depending on the front end implementation of Nimbus.

The properties of the action `push` are:
```typescript
interface PushProperties {
  url: string,
  method?: 'Get' | 'Patch' | 'Put' | 'Post' | 'Delete',
  data?: any,
  headers?: Record<string, string>,
  fallback?: Component,
  prefetch?: boolean,
  state?: Record<string, any>,
  events?: Record<string, Actions>
}
```

Where:
- `url` is the address to send the request to. If this url is relative, i.e. if it starts with a "/", then it will be appended to the
`baseUrl`, which is a property that must be setup by the frontend lib and represents the URL of the backend server.
- `method` is the request method. Default is `'Get'`.
- `data` is the json data to send in the request body. This is only valid for Patch, Put and Post requests.
- `headers` is the map of headers to send in the request.
- `fallback` is a UI tree to render if the request fails. Read ["More on fallback"](#more-on-fallback) for details on this.
- `prefetch` is a boolean that, when true, makes the frontend application load the contents of the screen we wish to navigate to as soon as possible.
This way, when the button is pressed, the navigation is immediate. Be careful with this, setting every navigation to `prefetch: true` may result high
and unnecessary internet traffic for the user. If a prefetch request fails the navigation behaves like `prefetch` was false.
- `state` a map where each entry is a state for the next screen: the key is the state name and the value is the initial value for the state. This is
used to communicate values from the calling screen to the target screen. Read ["More on states and events"](#more-on-states-and-events) for an example
on this.
- `events` a map where each entry represents an event that can be triggered by the next screen: the key is the event name and the value is the list
of actions to run once the event is triggered. Each of these events declares an state where the name is the event name and the value is the value
passed when the event is triggered by the target screen. Events are used to communicate values from the target screen to the screen who called
it. Read ["More on states and events"](#more-on-states-and-events) for an example on this.

### Example
```json
{
  "_:action": "push",
  "properties": {
    "url": "/products"
  }
}
```

Suppose the navigation stack `['/home']`. The action above performs a navigation to the screen "$baseUrl/products" and after it completes, the
navigation stack becomes `['/home', '/products']`.

### More on `fallback`
When `fallback` is not provided and the request fails, Nimbus will render an error component. This error component is customizable by the user via the
configuration `errorComponent` of the library.

The error component will always receive the following properties:
```typescript
interface ErrorComponentProperties {
  error: Error,
  retry: () => void,
}
```

Where:
- `error` is the platform-specific error object that prevented the screen from rendering.
- `retry` is a function that, when called, retries the navigation.

### More on states and events
An example of navigation with states and events would be an "edition screen". Suppose we have a screen that lists to do notes and another that edits a
specific note. Once we click in a note in the list screen, we navigate to the edit screen. Once we finish editing the note, we run the event
"onSaveNote" to tell the list page the note must be updated. See the json snippets below:

**list.json**
```json
{
  "_:component":"forEach",
  "properties":{
    "items":"@{notes}",
    "key":"id"
  },
  "children":[
    {
      "_:component":"layout:touchable",
      "properties":{
        "onPress":[
          {
            "_:action":"push",
            "properties":{
              "url":"/edit.json",
              "state":{
                "note":"@{item}"
              },
              "events":{
                "onNoteSaved":[
                  {
                    "_:action":"setState",
                    "properties":{
                      "path":"item.title",
                      "value":"@{onNoteSaved.title}"
                    }
                  },
                  {
                    "_:action":"setState",
                    "properties":{
                      "path":"item.description",
                      "value":"@{onNoteSaved.description}"
                    }
                  }
                ]
              }
            }
          }
        ]
      },
      "children":[
        {
          "_:component":"layout:text",
          "properties":{
            "text":"@{item.title}: @{item.description}"
          }
        }
      ]
    }
  ]
}
```

edit.json
```json
{
  "_:component": "layout:column",
  "state": {
    "title": "@{note.title}",
    "description": "@{note.description}"
  },
  "children": [
    {
      "_:component": "myApp:textInput",
      "properties": {
        "label": "Title",
        "value": "@{title}",
        "onChange": [
          {
            "_:action": "setState",
            "properties": {
              "path": "title",
              "value": "@{onChange}"
            }
          }
        ]
      }
    },
    {
      "_:component": "myApp:textInput",
      "properties": {
        "label": "Description",
        "value": "@{description}",
        "onChange": [
          {
            "_:action": "setState",
            "properties": {
              "path": "description",
              "value": "@{onChange}"
            }
          }
        ]
      }
    },
    {
      "_:component": "myApp:button",
      "properties": {
        "text": "Save",
        "onPress": [
          {
            "_:action": "triggerViewEvent",
            "properties": {
              "event": "onNoteSaved",
              "value": "@{object('id', note.id, 'title', title, 'description', description)}"
            }
          }
        ]
      }
    }
  ]
}
```

## pop
This action pops one screen from the navigation stack, effectively returning to the previous screen. A platform specific transition might be played
depending on the front end implementation of Nimbus.

Differently than `push`, where the current view is kept in the stack and just hidden in the UI, `pop` removes the current view from the stack, i.e.
the current view is lost.

`pop` doesn't accept any property.

If the current screen is the first screen of the Nimbus navigation stack, then nothing happens.

### Example
```json
{
  "_:action": "pop",
}
```

Suppose the navigation stack `['/home', '/products']`. The action above performs a "back" navigation, i.e. it removes the current screen and shows the previous screen. After the navigation is performed, the navigation stack is `['/home']`.

## popTo
This action is similar to `pop`, but instead of removing just the current screen, it removes all screens from right to left until a condition is met.
If the condition is never met, then nothing happens.

The condition is an equality in the url used to load the screen. `popTo` has a single property and it is:
```typescript
interface PopToProperties {
  url: string,
}
```

`popTo` goes through the current navigation stack from right to left until it finds a screen that has been loaded using `url`. Once it finds the
screen, it removes every screen it passed through before finding it.

### Example
```json
{
  "_:action": "popTo",
  "properties": {
    "url": "/products"
  }
}
```

Suppose the navigation stack `['/home', '/products', '/product/1', '/cart', '/payment']`. The action above goes back to the screen `/products`,
removing all screens that came after it. After the navigation is performed, the stack is `['/home', '/products']`.

Suppose we had the navigation stack `['/home', '/product/1', '/cart', '/payment']` before executing the same action. In this case nothing would happen
because `/products` won't be found.

# Presentation actions
## present
This action works exactly like `push`, it has the same properties and they do the same thing. The only difference between these actions is that
`present` opens the new screen as a modal dialog, **in a new navigation stack**.

"in a new navigation stack" is a very important part of the definition of this action. It means that any `push`, `pop` or `popTo` after a `present`
will happen inside the modal, and not in the navigation stack that opened it. A `pop` right after a `present` does nothing because the new stack
will have a single screen. A `popTo` after a `present` will only look at the screens inside this new navigation stack.

A `present` may use a transition depending in the frontend implementation of Nimbus.

`present` can be used as many times as wanted. You can present a screen from a screen that has itself been presented.

### Example
```json
{
  "_:action": "present",
  "properties": {
    "url": "/product/1"
  }
}
```

Suppose the navigation stack `['/home', '/products']`. The action above adds a new navigation stack to the stack, with a single screen. After the
navigation is performed, the navigation stack is: `['/home', '/products', ['/product/1']]`.

## dismiss
This action works like `pop`, but for presented screens. It dismisses the screen that is currently being presented. In other words, it removes the
current navigation stack from the navigation stack that created it.

`dismiss` doesn't accept any property.

If `dismiss` is used without a previous `present`, nothing happens.

### Example
```json
{
  "_:action": "dismiss",
}
```

Suppose the navigation stack `['/home', '/products', ['/product/1', '/product/1/comments']]`. The action above removes the current navigation stack,
effectively going back to `/products`. After the navigation is performed, the navigation stack is: `['/home', '/products']`.

If the stack was `['/home']`, before the action ran, then nothing would've happened.
