> Article level: basic.

# Components
Components are the main structure in Nimbus. Nimbus use a tree to represent the UI and the components are the node of this tree.

A component in Nimbus is the same as a component in most UI frameworks. Examples of components are: text, text-input, tab-bar, radio-button,
select-menu, data-input, row, column, button, etc.

Imagine a UI with a column containing a welcoming text at the top and a button below it. It could be represented with the following structure.

- Column
  - Text
  - Button

Each of the nodes of the tree above is a component. To be able to render a UI tree declared in the backend in the frontend, Nimbus serializes the
UI tree into a JSON and deserializes it into the actual components before rendering it.

A Nimbus component is defined by the following structure:

```typescript
interface Component {
  '_:component': string,
  id?: string,
  properties?: Record<string, any>,
  state?: {
    id: string,
    value: any,
  },
  children?: Node[],
}
```

Where:
- `_:component`: required. The name that identifies the component. It must follow the format: "namespace:name". The only components that don't need a
namespace are the default components that come with any Nimbus implementation. Examples of component names: if, forEach, layout:text, material:button,
layout:column.
- `id`: a unique id for the node within a UI tree. When not specified, a random id is assigned by the frontend.
- `properties`: a map with the properties for the component. If this is a text, for instance, this could be "text: string", "fontSize: double",
"fontWeight: integer".
- `state`: declares a [state](state.md) visible to this node and its descendants.
- `children`: the nodes that are children of this node. When absent, null or an empty array, this node is a leaf.

# Using components
To create a UI with components, we use the backend. See the examples below for creating the UI described in the previous section: a column with a
welcoming text and a button to go to another screen.

```json
{
  "_:component": "layout:column",
  "children": [
    {
      "_:component": "layout:text",
      "properties": {
        "text": "Hello"
      },
    },
    {
      "_:component": "layout:text",
      "properties": {
        "text": "World"
      }
    }
  ]
}
```

The JSON above renders a column with two texts: "Hello" in the first line and "World" in the second line.

# Default components
The Nimbus specification declares 2 structural components, i.e. any implementation must have them. The components shipped with Nimbus are:

- [if, then, else](default-components/if.md);
- [forEach](default-components/for-each.md).

# Custom components
Just like [operations](operation.md) and [actions](actions.md), Nimbus must provide a way for the developer to register new components. Nimbus ships
with only a few structural components. To actually build a UI, you need to create your own components or at least use an additional lib that provides
such components.

New components must be implemented in the frontend and declared in the backend.

How a new component is registered for Nimbus is particular to each implementation. Please check the platform-specific documentation to know more.

# Read next
:point_right: [State](/state)
