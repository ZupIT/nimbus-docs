# sendRequest
Check the [specification](/specification/default-actions/send-request.md) for the full definition of this action.

## Properties
```typescript
interface SendRequestParams<SuccessResponse, ErrorResponse> {
  /**
   * The URL to send the request to.
   */
  url: Expression<string>,
  /**
   * The method of the request.
   *
   * @defaultValue `'Get'`
   */
  method?: Expression<'Get' | 'Patch' | 'Put' | 'Post' | 'Delete'>,
  /**
   * The headers to send with this request.
   */
  headers?: Expression<Record<string, string>>,
  /**
   * The body to send in this request. This is invalid for get requests.
   */
  data?: any,
  /**
   * An action factory. This needs to return the actions to run once the request succeeds.
   *
   * This function receives as parameter the state for the onSuccess event, i.e. a state containing the status,
   * statusText and data of the response.
   *
   * @example
   * ```typescript
   * sendRequest<User, Error>(
   *   // ...
   *   onSuccess: (response) => alert(`Username is ${response.get('data').get('name')}`)
   * )
   * ```
   */
  onSuccess?: (response: State<ResponseState<SuccessResponse>>) => Actions,
  /**
   * An action factory. This needs to return the actions to run once the request fails.
   *
   * This function receives as parameter the state for the onError event, i.e. a state containing the status,
   * statusText and data of the response.
   *
   * @example
   * ```typescript
   * sendRequest<User, string>(
   *   // ...
   *   onError: (response) => alert(`An error occurred. ${response.get('data')}`)
   * )
   * ```
   */
  onError?: (response: State<ErrorState<ErrorResponse>>) => Actions,
  /**
   * Actions to run when the request finished. This is run after onSuccess and onError.
   */
  onFinish?: Actions,
}

interface ResponseState<T> {
  status: number,
  statusText: string,
  data: T,
}

interface ErrorState<T> extends ResponseState<T> {
  /**
   * The message for the error object in case the request failed before it was sent.
   */
  message: string,
}
```

## Example
The `sendRequest` mostly used when submitting a form or initializing a screen. See below an example that fetches a product, shows the text
"Loading..." while the request is pending and replaces it with the product details when it completes.

```tsx
import { NimbusJSX } from '@zup-it/nimbus-backend-core'
import { sendRequest, log } from '@zup-it/nimbus-backend-core/actions'
import { If, Then, Else } from '@zup-it/nimbus-backend-core/components'
import { Column, Text } from '@zup-it/nimbus-backend-layout'
import { Screen, ScreenRequest } from '@zup-it/nimbus-backend-express'
import { Product } from '../model/product' // describing this interface is not important for this example

interface ProductDetailsRequest extends ScreenRequest {
  routeParams: {
    id: string,
  }
}

interface ProductState {
  data?: Product,
  isLoading: boolean,
}

export const ProductDetailsScreen: Screen<ProductDetailsRequest> = ({ routeParams: { id }}) => {
  const productState = createState<ProductState>('product', { isLoading: true })
  const product = productState.get('product')
  const isLoading = productState.get('isLoading')

  const loadProduct = sendRequest<Product>({
    url: `https://my-backend.com/product/${id}`,
    onSuccess: response => product.set(response.get('data')),
    onError: response => log(response.get('message')),
    onFinish: isLoading.set(false),
  })

  return (
    <Column state={productState} onInit={loadProduct}>
      <If condition={isLoading}>
        <Then><>Loading...</></Then>
        <Else>
          <Column>
            <Text>Name: {product.get('name')}</Text>
            <Text>Price: {product.get('price')}</Text>
            <Text>Description: {product.get('description')}</Text>
          </Column>
        </Else>
      </If>
    </Container>
  )
}
```

## Composing requests
In the previous example we used a local state and a `sendRequest` action to load a product and show its details. It
works just fine, but code-quality wise it can be improved. When building frontend applications we should never mix the
definition of network requests (url, method, headers, etc) with the screen structure. The example we gave voids this
principle. Instead, we should have a place where we declare our network functions separately, then, in the screen code,
we should just call them.

We recommend creating a directory called `network` to place these functions. Then, for each REST resource, a file with
all its requests should be created. For the previous example, we'd create the file `network/product.ts`.

The ideal code for the previous example would be:
```tsx
// ...
import { findProductById } from '../network/product'

// ...

export const ProductDetailsScreen: Screen<ProductDetailsRequest> = ({ routeParams: { id }}) => {
  // ...

  const loadProduct = findProductById({
    id,
    onSuccess: response => product.set(response.get('data')),
    onError: response => log(response.get('message')),
    onFinish: isLoading.set(false),
  })

  // ...
}
```

So we have to create and export the function `findProductById` in `network/product.ts`. In summary, we must divide the
`sendRequest` into two steps, the first, declared in the function `findProductById` provides the url and method, while
the second, in the screen itself, provides the id and callbacks.

Fortunately, Nimbus offers an API that makes it very easy to compose requests in two steps. See the example below:

```typescript
import { request } from '@zup-it/nimbus-backend-core/actions'
import { Product } from '../model/product' // describing this interface is not important for this example
import { baseUrl } from '../constants' // it's important to have the base url declared as a constant

// the more semantic properties for the new function, findProductById needs only an id, not the full url
interface GetByIdOptions {
  id: string, // if you want to also accept state values, don't forget to change this to Expression<string>
}

export const findProductById = request<Product>()
  .compose(({ id }: GetByIdOptions) => ({ url: `${baseUrl}/product/${id}`, method: 'Get' }))
```

Now you just need to call `findProductById` to create the `sendRequest` action you need. `findProductById` will require
the property `id`, that must be a `string`, as defined by `GetByIdOptions`. It will also accept any property accepted by
`sendRequest`, except the ones that have already been defined. In this case, it will accept `onSuccess`, `onError`,
`onFinish`, `headers` and `data`, but will not accept either `url` or `method`.

In summary, `request<Success, Error>()` creates a function that, when called, creates a `sendRequest<Success, Error>`
with the combined properties of both function calls. The TS typing system guarantees that the properties provided in
the first call aren't required again in the second call.
