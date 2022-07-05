# sendRequest
To make requests to REST APIs in Nimbus, we use the action `sendRequest`. The properties for this action are the following:

```typescript
interface SendRequestProperties {
  url: string,
  method?: 'Get' | 'Patch' | 'Put' | 'Post' | 'Delete',
  data?: any,
  headers?: Record<string, string>,
  onSuccess?: Action[],
  onError?: Action[],
  onFinish?: Action[],
}
```

Where:
- `url` is the address to send the request to. If this url is relative, i.e. if it starts with a "/", then it will be appended to the
`baseUrl`, which is a property that must be setup by the frontend lib and represents the URL of the backend server.
- `method` is the request method. Default is `'Get'`.
- `data` is the json data to send in the request body. This is only valid for Patch, Put and Post requests.
- `headers` is the map of headers to send in the request.
- `onSuccess` is an event triggered whenever the request succeeds with status code lower than 400. This event creates an implicit state called
`onSuccess` that contains the properties:
  - `status`: status code of the response (`number`).
  - `statusText`: the description of the response status (`string`).
  - `data`: the response data. If the response is a valid json, it will be parsed. Otherwise, it will be a string.
- `onError` is an event triggered whenever the request fails, i.e. whenever it's not sent or returns with a status code of 400 or greater. This
event creates an implicit state called `onError` that contains the same properties as the implicit state created by the event "onSuccess". The only
difference is that when the request fails before it's sent:
  - `status` is 0;
  - `statusText` is "Unable to send request";
  - `data` is not available;
  - `message` is an extra property containing the message corresponding to the error that prevented the request from being sent.
- `onFinish` is an event triggered after `onSuccess` or `onError` is executed. It doesn't create any implicit state.

## Example:
Suppose the url `https://my-api.com/products` returns an array with objects with the following structure (Product):
```typescript
interface Product {
  id: string,
  name: string,
  image: string,
  description: string,
  price: number,
}
```

Let's create a UI that shows a product list. This product list will be loaded by the `sendRequest` action.
```json
{
  "_:component": "layout:lifecycle",
  "state": {
    "id": "products",
    "value": []
  },
  "properties": {
    "onInit": [
      {
        "_:action": "sendRequest",
        "properties": {
          "url": "https://my-api.com/products",
          "onSuccess": [
            {
              "_:action": "setState",
              "properties": {
                "path": "products",
                "value": "@{onSuccess.data}"
              }
            }
          ],
          "onError": [
            {
              "_:action": "log",
              "properties": {
                "message": "Request failed with status @{onError.status}",
                "level": "Error"
              }
            }
          ]
        }
      }
    ]
  },
  "children": [
    {
      "_:component": "layout:column",
      "children": [
        {
          "_:component": "layout:text",
          "properties": {
            "text": "Products",
            "size": 22
          }
        },
        {
          "_:component": "forEach",
          "properties": {
            "items": "@{products}",
            "key": "id"
          },
          "children": [
            {
              "_:component": "layout:column",
              "children": [
                {
                  "_:component": "layout:text",
                  "properties": {
                    "text": "@{item.name}",
                    "size": 16
                  }
                },
                {
                  "_:component": "layout:remoteImage",
                  "properties": {
                    "url": "@{item.image}"
                  }
                },
                {
                  "_:component": "layout:text",
                  "properties": {
                    "text": "@{item.description} - @{item.price}",
                    "size": 16
                  }
                },
              ]
            }
          ]
        }
      ]
    }
  ]
}
```

The UI above first declares the state `product` and initializes it with `[]`. Then, it uses the component `layout:lifecycle` to attach an action
to the event `onInit`. this component is not part of the Nimbus specification, but is implemented by [this library](/layout/index.md). This component
just manages events related to the component lifecycle, `onInit` runs when the component is first rendered.

We tell the UI, after it's first rendered, it should make a request to fetch the list of products from the backend. If the request succeeds,
we store the result in the state `products`. If the request fails, we log an error message with the request status.

The visual elements are comprised by a title and a loop structure that goes over every item in the state `products` and create a set of components
based on the template (children of the component [`forEach`](../default-components/for-each.md)).