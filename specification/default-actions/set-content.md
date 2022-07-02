# setContent
This action changes the UI tree by adding a new component as a child of an existing component or replacing it entirely. The properties for the
`setContent` action are:

```typescript
interface SetStateProperties {
  id: string,
  value: Component,
  mode?: 'Prepend' | 'Append' | 'Replace' | 'ReplaceItself',
}
```

Where:
- `id` is the id of the node to replace if the mode is `'ReplaceItself'` or the id of the node to receive the new child otherwise.
- `value` is the component to be added to the UI tree.
- `mode` is the strategy to use for adding the node to the tree. The default value is `'Append'`. The available modes are:
  - `'Prepend'`: adds the new node (`value`) before the current set of children of the component identified by `id`, i.e. at the first position of
  the array.
  - `'Append'`: adds the new node (`value`) after the current set of children of the component identified by `id`, i.e. at the last position of
  the array.
  - `'Replace'`: replaces the current set of children of the component identified by `id` with the new node (`value`).
  - `'ReplaceItself'`: replaces the component identified by `id`, not its children, by the new node (`value`).

## Examples:
Consider the following tree as the currently rendered UI:

```json
{
  "_:component": "layout:column",
  "id": "root",
  "children": [
    {
      "_:component": "layout:text",
      "properties": {
        "text": "header"
      }
    },
    {
      "_:component": "layout:row",
      "id": "box",
      "children": [
        {
          "_:component": "layout:column",
          "id": "original"
        }
      ]
    },
    {
      "_:component": "layout:text",
      "properties": {
        "text": "footer"
      }
    },
  ]
}
```

#### Add a new component to "box", before the current content:
```json
{
  "_:action": "setContent",
  "properties" {
    "id": "box",
    "mode": "Prepend",
    "value": {
      "_:component": "layout:column",
      "children": [
        {
          "_:component": "layout:text",
          "properties": {
            "text": "Hello World!"
          }
        }
      ]
    }
  },
}
```
**Result:**
```json
{
  "_:component": "layout:column",
  "id": "root",
  "children": [
    {
      "_:component": "layout:text",
      "properties": {
        "text": "header"
      }
    },
    {
      "_:component": "layout:row",
      "id": "box",
      "children": [
        {
          "_:component": "layout:column",
          "children": [
            {
              "_:component": "layout:text",
              "properties": {
                "text": "Hello World!"
              }
            }
          ]
        },
        {
          "_:component": "layout:column",
          "id": "original"
        }
      ]
    },
    {
      "_:component": "layout:text",
      "properties": {
        "text": "footer"
      }
    },
  ]
}
```

#### Add a new component to "box", after the current content:
```json
{
  "_:action": "setContent",
  "properties" {
    "id": "box",
    "mode": "Append",
    "value": {
      "_:component": "layout:column",
      "children": [
        {
          "_:component": "layout:text",
          "properties": {
            "text": "Hello World!"
          }
        }
      ]
    }
  },
}
```
**Result:**
```json
{
  "_:component": "layout:column",
  "id": "root",
  "children": [
    {
      "_:component": "layout:text",
      "properties": {
        "text": "header"
      }
    },
    {
      "_:component": "layout:row",
      "id": "box",
      "children": [
        {
          "_:component": "layout:column",
          "id": "original"
        },
        {
          "_:component": "layout:column",
          "children": [
            {
              "_:component": "layout:text",
              "properties": {
                "text": "Hello World!"
              }
            }
          ]
        }
      ]
    },
    {
      "_:component": "layout:text",
      "properties": {
        "text": "footer"
      }
    },
  ]
}
```

#### Replace the current content of "box":
```json
{
  "_:action": "setContent",
  "properties" {
    "id": "box",
    "mode": "Replace",
    "value": {
      "_:component": "layout:column",
      "children": [
        {
          "_:component": "layout:text",
          "properties": {
            "text": "Hello World!"
          }
        }
      ]
    }
  },
}
```
**Result:**
```json
{
  "_:component": "layout:column",
  "id": "root",
  "children": [
    {
      "_:component": "layout:text",
      "properties": {
        "text": "header"
      }
    },
    {
      "_:component": "layout:row",
      "id": "box",
      "children": [
        {
          "_:component": "layout:column",
          "children": [
            {
              "_:component": "layout:text",
              "properties": {
                "text": "Hello World!"
              }
            }
          ]
        }
      ]
    },
    {
      "_:component": "layout:text",
      "properties": {
        "text": "footer"
      }
    },
  ]
}
```

#### Replace the current content of "box":
```json
{
  "_:action": "setContent",
  "properties" {
    "id": "box",
    "mode": "Replace",
    "value": {
      "_:component": "layout:column",
      "children": [
        {
          "_:component": "layout:text",
          "properties": {
            "text": "Hello World!"
          }
        }
      ]
    }
  },
}
```
**Result:**
```json
{
  "_:component": "layout:column",
  "id": "root",
  "children": [
    {
      "_:component": "layout:text",
      "properties": {
        "text": "header"
      }
    },
    {
      "_:component": "layout:row",
      "id": "box",
      "children": [
        {
          "_:component": "layout:column",
          "children": [
            {
              "_:component": "layout:text",
              "properties": {
                "text": "Hello World!"
              }
            }
          ]
        }
      ]
    },
    {
      "_:component": "layout:text",
      "properties": {
        "text": "footer"
      }
    },
  ]
}
```

#### Replace the component "box" itself:
```json
{
  "_:action": "setContent",
  "properties" {
    "id": "box",
    "mode": "ReplaceItself",
    "value": {
      "_:component": "layout:column",
      "children": [
        {
          "_:component": "layout:text",
          "properties": {
            "text": "Hello World!"
          }
        }
      ]
    }
  },
}
```
**Result:**
```json
{
  "_:component": "layout:column",
  "id": "root",
  "children": [
    {
      "_:component": "layout:text",
      "properties": {
        "text": "header"
      }
    },
    {
      "_:component": "layout:column",
      "children": [
        {
          "_:component": "layout:text",
          "properties": {
            "text": "Hello World!"
          }
        }
      ]
    },
    {
      "_:component": "layout:text",
      "properties": {
        "text": "footer"
      }
    },
  ]
}
```