# Default components
The Nimbus specification defines only two components as default, i.e., only these two components must be implemented when implementing Nimbus for a
new platform. They are:

- `If/Then/Else`: although these are three components, they act as one, since `Then` and `Else` are companions to `If` and can't be used anywhere
else. This is a structural component, i.e. it doesn't result in a UI component printed in the screen, it actually changes the tree structure before
it gets rendered so only the node we want to render ends up in the screen.
- `ForEach`: this is also a structural component, it doesn't directly translate to a UI component in the screen, instead it loops over an array and
use a template to create the actual UI components.

To know more about the default components, please read the [dedicated topic](/default-components.md).