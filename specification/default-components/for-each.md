# forEach
Just like `if`, this is a structural component, it doesn't directly translate into a UI component in the screen, instead it loops over an array and
uses a template to create the actual UI components.

`forEach` loops over its items and create a UI based on the template declared by its children. Whenever the array of items change, this component
must respond by updating the UI accordingly.

`forEach` must not recreate every component when the array of items change. Only new items can be created. Items that already exist, must be updated
only. To identify a component in the array, we must use the property `key` of the component `forEach`. `key` identifies which property in each item
is unique and can be used to identify it. If no `key` is provided, the components are identified by their position in the array of items.

Components inside a template have ids just like any other node in the tree. When `forEach` repeats the template it must alter the ids so it doesn't
generate duplicates. `forEach` appends `:$key_value` to the id of every node in the template, where `$key_value` is the value of the property `key`.
If `key` isn't specified, it uses the current iteration index.

## Properties
```typescript
interface ForEachProperties {
  items?: any[],
  iteratorName?: string,
  indexName?: string,
  key?: string,
}
```

Where:
- `items` is the array of items to iterate over.
- `iteratorName` is the name of the implicit state created to contain the current item of the iteration. Default is `item`.
- `indexName` is the name of the implicit state created to contain the index of the current iteration. Default is `index`.
- `key` is the name of the property of each item in `items` that can be used to identify it. By default, `forEach` uses the current index of the
iteration. It is a best practice to always specify a key. Specifying a key will make the rendering faster and prevent undesired behavior.

## Example
```json
{
  "_:component": "layout:column",
  "state": {
    "users": {
      { "id": "u003", "name": "John", "age": 30 },
      { "id": "u009", "name": "Mary", "age": 22 },
      { "id": "u055", "name": "Anthony", "age": 5 }
    }
  },
  "children": [
    {
      "_:component": "layout:text",
      "properties": {
        "text": "Here's all the people in the database:"
      }
    },
    {
      "_:component": "layout:column",
      "children": [
        {
          "_:component": "forEach",
          "properties": {
            "items": "@{users}",
            "key": "id"
          },
          "children": [
            {
              "_:component": "layout:text",
              "id": "index",
              "properties": {
                "text": "@{index}"
              }
            },
            {
              "_:component": "layout:text",
              "id": "name",
              "properties": {
                "text": "@{item.name}"
              }
            },
            {
              "_:component": "layout:text",
              "id": "age",
              "properties": {
                "text": "@{item.age}"
              }
            }
          ]
        }
      ]
    }
  ]
}
```

The code above, will render the following UI tree:

```json
{
  "_:component": "layout:column",
  "state": {
    "users": {
      { "id": "u003", "name": "John", "age": 30 },
      { "id": "u009", "name": "Mary", "age": 22 },
      { "id": "u055", "name": "Anthony", "age": 5 }
    }
  },
  "children": [
    {
      "_:component": "layout:text",
      "properties": {
        "text": "Here's all the people in the database:"
      }
    },
    {
      "_:component": "layout:column",
      "children": [
        {
          "_:component": "layout:text",
          "id": "index:u003",
          "properties": {
            "text": "0"
          }
        },
        {
          "_:component": "layout:text",
          "id": "name:u003",
          "properties": {
            "text": "John"
          }
        },
        {
          "_:component": "layout:text",
          "id": "age:u003",
          "properties": {
            "text": "30"
          }
        },
        {
          "_:component": "layout:text",
          "id": "index:u009",
          "properties": {
            "text": "1"
          }
        },
        {
          "_:component": "layout:text",
          "id": "name:u009",
          "properties": {
            "text": "Mary"
          }
        },
        {
          "_:component": "layout:text",
          "id": "age:u009",
          "properties": {
            "text": "22"
          }
        },
        {
          "_:component": "layout:text",
          "id": "index:u055",
          "properties": {
            "text": "2"
          }
        },
        {
          "_:component": "layout:text",
          "id": "name:u055",
          "properties": {
            "text": "Anthony"
          }
        },
        {
          "_:component": "layout:text",
          "id": "age:u055",
          "properties": {
            "text": "5"
          }
        }
      ]
    }
  ]
}
```

If the ids hadn't come from the backend, the strings (`:$key_value`) would've been appended to the ids generated in the frontend.
