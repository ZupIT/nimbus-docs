# Giving a type to your screens
## Contents
1. [Introduction](#introduction)
1. [The ScreenRequest type](#the-screenrequest-type)
1. [Deciding which components to use based on the platform](#deciding-which-components-to-use-based-on-the-platform)
1. [Keep reading](#keep-reading)

## Introduction
In Typescript, it is always recommended to give a type to your variables. Screens, should always use the type `Screen`.
See the example below:

Instead of writing:
```jsx
import { NimbusJSX } from '@zup-it/nimbus-backend-core'

const MyScreen = () => <>Hello from my first screen!</>
```

prefer:
```jsx
import { NimbusJSX } from '@zup-it/nimbus-backend-core'
import { Screen } from '@zup-it/nimbus-backend-express'

const MyScreen: Screen = () => <>Hello from my first screen!</>
```

## The ScreenRequest type
The type `Screen` is generic, by default it will be a `Screen` of `ScreenRequest`, which means a screen will always
receive as parameter the following data:

- `request`: [the request object from express](https://expressjs.com/en/4x/api.html#req) used to request this page.
Use this to get the request headers, route parameters, query, body, etc.
- `response`: [the response object from express](https://expressjs.com/en/4x/api.html#res) for the current response.
Use this to alter the status code of the response, headers, etc.
- `getViewState`: a function that gets a state linked to the current view. View states are created when a parameter
is passed while navigating.
- `navigator`: an instance of the class Navigator, which is a type-safe way of navigating from a screen to another.
Always prefer using the navigator instead of the navigation actions directly from the backend-ts core. To know more
about the Navigator, check [this topic](default-actions/navigation.md#the-navigator).

With these properties injected into your screen you can get lots of information from the request and also alter the
response! But which route params should the screen expect? What about the query params and view states? And what's 
the type of the body anyway?

If your screen expects anything specific in the request, you should type it accordingly. Take as example a screen
that shows a product with a given id. One way of receiving this id is through the url, to do so, extend the type
`ScreenRequest` and set it as the generic value for `Screen`. See the example below:

```tsx
import { NimbusJSX } from '@zup-it/nimbus-backend-core'
import { Screen, ScreenRequest } from '@zup-it/nimbus-backend-express'

interface ProductRequest extends ScreenRequest {
  routeParams: {
    id: string,
  }
}

const ProductScreen: Screen<ProductRequest> = ({ request }) => <>The product id is: {request.params.id}</>
```

And when registering the screen, don't forget to express that the route accepts the id as parameter:

```typescript
// ...
import { RouteMap } from '@zup-it/nimbus-backend-express'
import { ProductScreen } from './product'

export const routes: RouteMap = {
  // ...
  '/product/:id': ProductScreen,
}
```

In the example above we typed the route parameters, but we can type any part of the request! Check the example below:

```tsx
interface MyScreenRequest extends ScreenRequest {
  routeParams: {
    param1: string,
    param2: string,
  },
  query?: {
    queryParam1?: string,
    queryParam2?: string,
  },
  headers: {
    'my-custom-header': string,
  },
  body: {
    attribute1: number,
    attribute2: boolean,
    attribute3: {
      attribute4: string,
    }
  },
  params: {
    data1: string,
    data2: number,
  }
}

const MyScreen: Screen<ProductRequest> = ({ request, getViewState }) => (
  <>
    <>{request.params.param1}</>
    <>{request.params.param2}</>
    <>{request.query?.queryParam1 ?? ''}</>
    <>{request.query?.queryParam2 ?? ''}</>
    <>{request.body.attribute1}</>
    <>{request.body.attribute2}</>
    <>{request.body.attribute3.attribute4}</>
    <>{getViewState('data1')}</>
    <>{getViewState('data2')}</>
  </>
)
```

This example doesn't represent any real use-case scenario, it is just to show how we can type every aspect of the
request. This is very important so, inside the screen function, we can access only what actually exists. It is also
important to guarantee that any navigation to this screen won't miss any of the required parameters.

## Deciding which components to use based on the platform
The `Screen` receives `request.headers` as parameter and Nimbus expects the header `nimbus-platform`, which is a string.

We can use this header to decide which components or properties to display. See the example below:

```tsx
import { NimbusJSX } from '@zup-it/nimbus-backend-core'
import { Screen } from '@zup-it/nimbus-backend-express'
import { Colum, LocalImage, Text } from '@zup-it/nimbus-backend-layout'
import { Button } from '../components/button' // this component's implementation is not important for this example

const MyScreen: Screen = ({ request: { headers } }) => {
  const content = (
    <>
      <Text>This is the common content between all platforms</Text>
      <Button>Ok!</Button>
   </>
  )

  return headers['nimbus-platform'] == 'swiftui' ? (
    <Container style={{ flexDirection: 'ROW' }}>
      <Image type="remote" url="https://imagebank/logo-ios.png" style={ { marginRight: 10 } } />
      {content}
    </Container>
  ) : (
    <>
      <LocalImage id="logo-android" marginBottom={10} />
      {content}
    </>
  )
}
```

Now, the screen will be different based on the platform that requested it. The platform string is decided by the frontend lib.

## Read next
:point_right: [Components](component.md)
