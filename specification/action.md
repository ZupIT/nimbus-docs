> Article level: basic.

# Actions
Actions are the third and final Nimbus feature that allows views to be dynamic. Just like operations, actions can be
viewed as functions, but with an important difference: while operations always have a return value and never have
collateral effects (it only uses its parameters to calculate the return value and don't alter anything outside its
scope), actions never have a return value and always have collateral effects.

It's true one could implement a similar behavior to an action using an operation. But this would be wrong and is highly
discouraged. Use operations to transform a state value into something else that is needed as an input to a component
or an action. Use actions to trigger behaviors like opening a dialog, playing a sound, sending a request, etc.

Actions are side-effects executed by the frontend when a given event occurs. For instance, when a
button is pressed (event) we might want to navigate to another screen (action). Actions are always associated to events.
Example of events are:

- `onChange` in a text input or any other form element;
- `onTabChange` in a component that displays a tabbed view;
- `onPullToRefresh` in a scroll view with the pull to refresh behavior;
- `onInit` in a lifecycle component;
- `onScrollEnd` in a list view with multiple elements;
- `onSuccess` in an action that sends a request;
- `onPressOk` in a dialog box.

There are many events that could be though of and they're always related to Components or Actions. Any set
of actions can be assigned to any event.

# Action structure
A component property may be a list of [actions]. Actions are identified by the key `_:action` and its value must
identify which action to run. See the format of an Action object below:

```typescript
interface Action {
  '_:action': string,
  properties?: Record<string, any>,
  metadata?: Record<string, any>,
}
```

Just like the components, actions have a single required parameter: `_:action`. See below the description of each property in an action object:
- `_:action`: required. The name that identifies the action. It must follow the format: "namespace:name". The only actions that don't need a
namespace are the default actions that come with any Nimbus implementation. Examples of actions: sendRequest, log, setState, material:alert,
material:confirm.
- `properties`: a map of properties for the action. The structure of this map will vary depending on the value of `_:action`. A log instruction, for
instance, will have a completely different set of properties when compared to a an instruction to send a request or to change a state value.
- `metadata`: this is an advanced feature that allows us to pass data that won't be used by the action itself, but might be useful for logging data or
creating analytics records.

# Example
```json
{
  "_:component": "material:button",
  "properties": {
    "text": "click me",
    "onPress": [{
      "_:action": "log",
      "properties": {
        "message": "The button has been clicked!"
      }
    }]
  }
}
```

In the example above, a button is rendered and when it's pressed, it logs a message to the console. We say that "onPress" is the event name and its
value is the list of actions to execute when the event occurs. Notice that the value is an array. Event values must always be arrays, if a single
action must run, the array will a have a single Action.

# Default Actions
Nimbus ships with some basic actions, they are:

- [Navigation actions](default-actions/navigation.md)
- [condition](default-actions/condition.md)
- [log](default-actions/log.md)
- [openUrl](default-actions/open-url.md)
- [sendRequest](default-actions/send-request.md)
- [setContent](default-actions/log.md)
- [setState](default-actions/set-state.md)

# Custom actions
Just like [components](/component.md) and [operations](/operation.md), Nimbus must provide a way for the developer to register new actions. Custom
actions are very useful for doing things like opening dialogs, performing asynchronous tasks, playing animations or sounds, and much more.

How a new action is registered for Nimbus is particular to each implementation. Please check the platform-specific documentation to know more.

# Read next
:point_right: [TODO](/todo_link)
