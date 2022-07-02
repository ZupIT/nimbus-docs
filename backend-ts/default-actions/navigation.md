# Navigation actions
Please, read the [specification](/specification/default-actions/navigation.md) for a detailed explanation of each navigation action.

# Using navigation actions
All navigation actions are available under `@zup-it/nimbus-backend-core/actions`, but if you're using `nimbus-backend-express`, there's a better way
to navigate. Any Screen in a Nimbus application with Express receives the `navigator` as a property, which is type-safe way of navigating. If you're
not using `nimbus-backend-express`, skip to the section ["Using navigation actions without Express"](#using-navigation-actions-without-express).

## The Navigator
Each screen receives the Navigator as one of its properties and the Navigator can perform any of the navigation actions, i.e. `push`, `pop`, `popTo`,
`present` or `dismiss`. The only difference from directly using the action is that, instead of dealing with strings, it uses the screens themselves to
navigate. This ensures we never use an incorrect URL or try to navigate to a screen that doesn't exist. Furthermore, the Navigator ensures that
everything expected by a screen is correctly passed when we try to navigate to it.

The example below shows two screens: the first is a page that displays the details of a product. The product's id is expected
to be passed as a route parameter. The second is a screen with 3 buttons representing each one a product. When
the button is pressed, it navigates to the screen that shows its details (in this example, just the id).

`product.tsx`
```tsx
import { NimbusJSX } from '@zup-it/nimbus-backend-core'
import { Screen, ScreenRequest } from '@zup-it/nimbus-backend-express'
import { Column, Text } from '@zup-it/nimbus-backend-layout'
import { Product } from '../model/product' // no need to show this interface to explain this topic

interface ProductScreenRequest extends ScreenRequest {
  routeParams: {
    productId: string,
  }
}

export const ProductScreen: Screen<ProductScreenRequest> = ({ navigationContext }) => (
  <Column>
    <Text>{navigationContext.get('productId')}</Text>
  </Column>
)
```

`product-list.tsx`
```tsx
import { NimbusJSX } from '@zup-it/nimbus-backend-core'
import { Screen } from '@zup-it/nimbus-backend-express'
import { Column } from '@zup-it/nimbus-backend-layout'
import { Product } from '../model/product' // no need to show this interface to explain this topic
import { Button } from '../components/button' // the component declaration is not important for this example
import { ProductScreen } from './product'

const products: Product[] = [
  // ... (suppose we declared 3 products here)
]

export const ProductListScreen: Screen = ({ navigator }) => (
  <Column>
    <Button onPress={navigator.push(ProductScreen, { routeParams: { productId: products[0].get('id') } })}>Product 1</Button>
    <Button onPress={navigator.push(ProductScreen, { routeParams: { productId: products[1].get('id') } })}>Product 2</Button>
    <Button onPress={navigator.push(ProductScreen, { routeParams: { productId: products[2].get('id') } })}>Product 3</Button>
  </Column>
)
```

Notice that we imported the screen from the file we declared it and when navigating to it, we specified the product
it expects in the route parameters. If we hadn't passed the `productId`, this code wouldn't compile, because when we
created the `ProductScreen`, we said it needed `productId` as a route parameter.

An interface that extends `ScreenRequest` can specify the following properties:
```typescript
interface ScreenRequest {
  /**
   * The request headers expected to be in the request.
   *
   * Example: if you expect a header called "session-token", this should be:
   * ```typescript
   * { 'session-token': string }
   * ```
   */
  headers?: Record<string, string>,
  /**
   * The parameters expected in the route.
   *
   * Example: if the route is "/user/:userId/books/:bookId", this should be:
   * ```typescript
   * {
   *   userId: string,
   *   bookId: string,
   * }
   * ```
   */
  routeParams?: Record<string, string>,
  /**
   * The query parameters expected in the route.
   *
   * Example: if the route is "/list?order=crescent&page=1&limit=10", this should be:
   * ```typescript
   * {
   *   order: 'crescent' | 'decrescent',
   *   page: `${number}`,
   *   limit: `${number}`,
   * }
   * ```
   */
  query?: Record<string, string>,
  /**
   * The type of the request body. If a JSON is expected, specify here the object interface.
   */
  body?: unknown,
  /**
   * The type of the navigation context of this screen. If it's an order page and you expect to receive both the order
   * id and the address from the navigation context, this should be:
   * ```typescript
   * {
   *   orderId: string,
   *   address: Address,
   * }
   * ```
   */
  navigationContext?: unknown,
}
```

When any of these properties are set in the screen interface, they're required when trying to navigate to the screen.

### pop, popTo, present and dismiss
`present` works just like the previous example (`push`).

`pop` and `dismiss` don't accept any arguments.

`popTo` accepts a single argument, the screen object. See the example below, where we add a button to the product page that returns to the home
screen:
`product.tsx`
```tsx
import { NimbusJSX } from '@zup-it/nimbus-backend-core'
import { Screen, ScreenRequest } from '@zup-it/nimbus-backend-express'
import { Column, Text } from '@zup-it/nimbus-backend-layout'
import { HomeScreen } from './home' // screen for the home page
import { Product } from '../model/product' // no need to show this interface to explain this topic
import { Button } from '../components/button' // the component declaration is not important for this example

// ...

export const ProductScreen: Screen<ProductScreenRequest> = ({ navigationContext, navigator }) => (
  <Column>
    <Text>{navigationContext.get('productId')}</Text>
    <Button onPress={navigator.popTo(HomeScreen)}>Go Back to home</Button>
  </Column>
)
```

# Using navigation actions without Express
If you're not using `nimbus-backend-express` you won't have access to the Navigator and must access the navigation actions directly. To do so, import
them from `@zup-it/nimbus-backend-core/actions`.

## push, pop and popTo
### Properties
```typescript
interface PushParams {
  /**
   * The URL of the screen to fetch.
   */
  url: Expression<string>,
  /**
   * The HTTP method to use when fetching the screen.
   *
   * @defaultValue `'Get'`
   */
  method?: 'Get' | 'Patch' | 'Put' | 'Post' | 'Delete',
  /**
   * The headers to send with the request.
   */
  headers?: Record<string, string>,
  /**
   * The body to send with the request. Invalid for GET requests.
   */
  body?: any,
  /**
   * When set to true, the frontend application will load this as soon as possible instead of waiting the navigation
   * action to be triggered.
   */
  prefetch?: boolean,
  /**
   * Component tree to show if the screen can't be fetched.
   */
  fallback?: Component,
}

interface PopToParams {
  /**
   * The URL of the screen to go back to.
   */
  url: Expression<string>,
}
```

### Example
```tsx
import { NimbusJSX } from '@zup-it/nimbus-backend-core'
import { push, pop, popTo } from '@zup-it/nimbus-backend-core/actions'
import { Screen } from '@zup-it/nimbus-backend-express'
import { Column } from '@zup-it/nimbus-backend-layout'
import { Product } from '../model/product' // no need to show this interface to explain this topic
import { Button } from '../components/button' // the component declaration is not important for this example

const products: Product[] = [
  // ... (suppose we declared 3 products here)
]

export const ProductListScreen: Screen = ({ navigator }) => (
  <Column>
    <Button onPress={push(`/product/${products[0].get('id')}`)}>Product 1</Button>
    <Button onPress={push({ url: `/product/${products[1].get('id')}`, method: 'Get' })}>Product 2</Button>
    <Button onPress={push(`/product/${products[2].get('id')}`)}>Product 3</Button>
    <Button onPress={pop()}>Go back</Button>
    <Button onPress={popTo('/home')}>Go back to Home</Button>
  </Column>
)
```

## present and dismiss
`present` works just like `push`, just import `present` instead of `push`.
`dismiss` works just like `pop`, just import `dismiss` instead of `pop`.
