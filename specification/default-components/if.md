# if, then, else
Although these are three components, they act as one, since `then` and `else` are companions to `if` and can't be used anywhere
else. This is a structural component, i.e. it doesn't result in a UI component printed in the screen, it actually changes the tree structure before
it gets rendered so only the node we want to render ends up in the screen.

If the `condition` passed to the component `if` is `true`, the children of `then` are rendered. Otherwise, the children of `else` are rendered.

`if` requires `then` as a child. `else` is optional and when not provided, if the condition is false, nothing gets rendered.

`if` doesn't accept any child other than `then` and `else`.

The content rendered by `if` must update according to the state passed to `condition`, i.e. if `condition` changes from `true` to `false` or from
`false` to `true`, the rendered content should change accordingly.

> Attention: `if` controls which components end up in the rendered UI tree. Sometimes, instead of altering the tree, the developer needs to keep the
tree intact and just show or hide a component based on a property. For this behavior, we advise creating a custom component. If you're familiar
with web development, `if` would be equivalent to set `display: none` instead of `height: 0` and/or `opacity: 0`. This distinction generally makes
a difference only when you need to play animations (because the element must exist while the animation is playing).

## Properties
```typescript
interface IfProperties {
  condition: boolean,
}
```

Where:
- `condition` indicates whether to render the contents of `then` (true) or the contents of `else` (false).

The components `else` and `then` can't receive any property.

## Example
```json
{
  "_:component": "layout:column",
  "state": {
    "id": "isMorning"
  },
  "children": [
    {
      "_:component": "if",
      "properties": {
        "condition": "@{isMorning}"
      },
      "children": [
        {
          "_:component": "then",
          "children": [
            {
              "_:component": "material:text",
              "properties": {
                "text": "Good morning!"
              }
            },
            {
              "_:component": "layout:localImage",
              "properties": {
                "id": "sun"
              }
            }
          ]
        },
        {
          "_:component": "else",
          "children": [
            {
              "_:component": "material:text",
              "properties": {
                "text": "Good evening!"
              }
            },
            {
              "_:component": "layout:localImage",
              "properties": {
                "id": "moon"
              }
            }
          ]
        }
      ]
    }
  ]
}
```

In the example above, we left the state `isMorning` without a value on purpose.

Suppose we set the value of `isMorning` to `true`. In this case, the UI rendered is the children of `then`, i.e. the UI tree that ends up in the
screen is the following:

```json
{
  "_:component": "layout:column",
  "state": {
    "id": "isMorning"
  },
  "children": [
    {
      "_:component": "material:text",
      "properties": {
        "text": "Good morning!"
      }
    },
    {
      "_:component": "layout:localImage",
      "properties": {
        "id": "sun"
      }
    }
  ]
}
```

Now suppose we set the value of `isMorning` to `false`. In this case, the UI rendered is the children of `else`, i.e. the UI tree that ends up in the
screen is the following:

```json
{
  "_:component": "layout:column",
  "state": {
    "id": "isMorning"
  },
  "children": [
    {
      "_:component": "material:text",
      "properties": {
        "text": "Good evening!"
      }
    },
    {
      "_:component": "layout:localImage",
      "properties": {
        "id": "moon"
      }
    }
  ]
}
```
