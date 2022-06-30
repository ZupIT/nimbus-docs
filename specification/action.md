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

Just like the components, actions have a single required parameter: `_:action`. The properties are:
- `_:action`: required. The name that identifies the action. It must follow the format: "namespace:name". The only actions that don't need a
namespace are the default actions that come with any Nimbus implementation. Examples of actions: sendRequest, log, setState, material:alert,
material:confirm.
- `properties`: a map of properties for the action. The structure of this map will vary depending on the value of `_:action`. A log instruction, for
instance, will have a completely different set of properties when compared to a an instruction to send a request or to change a state value.
- `metadata`: this is an advanced feature that allows us to pass data that won't be used by the action itself, but might be useful for logging data or
creating analytics records.

# Default Actions
Nimbus ships with some important actions already. Below you can find a list with all of them and links to their
respective API docs.

- `state.set` (`setState`) [[docs](State#writing-to-a-state) | [api](https://zupit.github.io/nimbus-backend-ts/classes/_zup_it_nimbus_backend_core.StateNode.html#set)]
- [`alert`](https://zupit.github.io/nimbus-backend-ts/modules/_zup_it_nimbus_backend_core_actions.html#alert);
- [`confirm`](https://zupit.github.io/nimbus-backend-ts/modules/_zup_it_nimbus_backend_core_actions.html#confirm);
- [`conditionalAction`](https://zupit.github.io/nimbus-backend-ts/modules/_zup_it_nimbus_backend_core_actions.html#conditionalAction);
- `sendRequest` [[docs](Making-requests) | [api](https://zupit.github.io/nimbus-backend-ts/modules/_zup_it_nimbus_backend_core_actions.html#sendRequest)];
- [`addChildren`](https://zupit.github.io/nimbus-backend-ts/modules/_zup_it_nimbus_backend_core_actions.html#addChildren);
- Navigation (check [[this topic|Navigation]] for more details on navigation actions):
  - [`openNativeRoute`](https://zupit.github.io/nimbus-backend-ts/modules/_zup_it_nimbus_backend_core_actions.html#openNativeRoute);
  - [`openExternalUrl`](https://zupit.github.io/nimbus-backend-ts/modules/_zup_it_nimbus_backend_core_actions.html#openExternalUrl);
  - [`pushStack`](https://zupit.github.io/nimbus-backend-ts/modules/_zup_it_nimbus_backend_core_actions.html#pushStack);
  - [`popStack`](https://zupit.github.io/nimbus-backend-ts/modules/_zup_it_nimbus_backend_core_actions.html#popStack);
  - [`pushView`](https://zupit.github.io/nimbus-backend-ts/modules/_zup_it_nimbus_backend_core_actions.html#pushView);
  - [`popView`](https://zupit.github.io/nimbus-backend-ts/modules/_zup_it_nimbus_backend_core_actions.html#popView);
  - [`popToView`](https://zupit.github.io/nimbus-backend-ts/modules/_zup_it_nimbus_backend_core_actions.html#popToView);
  - [`resetStack`](https://zupit.github.io/nimbus-backend-ts/modules/_zup_it_nimbus_backend_core_actions.html#resetStack);
  - [`resetApplication`](https://zupit.github.io/nimbus-backend-ts/modules/_zup_it_nimbus_backend_core_actions.html#resetApplication);
- [`submitForm`](https://zupit.github.io/nimbus-backend-ts/modules/_zup_it_nimbus_backend_components.html#submitForm) (available in `@zup-it/nimbus-backend-components`).

## Creating your own actions
It's very simple to create custom Actions. The examples below show how to create an action called "notify". The idea
is that whenever "notify" is called, a floating message appears in the screen and disappears after a while.

```typescript
import { Expression, Actions, WithAnalytics } from '@zup-it/nimbus-backend-core'
import { Action } from '@zup-it/nimbus-backend-core/model/action' // not recommended, we'll see another approach later

interface Notify extends WithAnalytics {
  /**
   * The message of the notification.
   */
  message: Expression<string>,
  /**
   * The title to show in the box.
   */
  title?: Expression<string>,
  /**
   * The icon to show at the left of the notification.
   */
  icon?: Expression<string>,
  /**
   * The time in ms to the message disappear.
   *
   * @defaultValue `300`
   */
  duration?: Expression<number>,
  /**
   * The actions to run once the user presses the notification message.
   */
  onPress?: Actions,
}

/**
 * Creates a floating notification with a message that will disappear after some time.
 *
 * @param properties. See {@link Notify}.
 */
export function notify({ message, title, icon, onPress, analytics }: Notify) {
  return new Action({
    name: 'notify',
    properties: { message, title, icon, onPress },
    analytics,
  })
}
```

First, notice that we make a strange import of the class `Action`. This is not recommended, but it's useful for this
example, in the next we'll show a better way of achieving the same result.

Every Nimbus Action supports analytics and that's why we make `Notify` extend `WithAnalytics`. To know more about the
Analytics feature, check [[this topic|Analytics]].

The Action factory is the function `notify`. It receives the parameters needed by the action and returns an instance of
`Action`. To use it, the developer only needs to import it and call `notify({ message: 'Test', title: 'Hey!' })`.

Every action factory will have basically the same code. So, to avoid repetition, instead of instantiating `Action`, you
should use the function `createAction` that does all of the code above under the hood. In fact, to prevent inefficient
usage, Nimbus doesn't even export the class `Action` (that's why we needed that weird deep import). See in the example
below the recommended way of creating actions:

```typescript
import { Expression, Actions, createAction } from '@zup-it/nimbus-backend-core'

interface Notify {
  message: Expression<string>,
  title?: Expression<string>,
  icon?: Expression<string>,
  duration?: Expression<number>,
  onPress?: Actions,
}

export const notify = createAction<Notify>('notify')
```

Notice that `Notify` doesn't extend `WithAnalytics` anymore. The `createAction` function knows you're creating an action
and will take care of this for you.

This second example is much more simple than the first and does the exact same thing. The only difference is that most
of the code of the first example is now encapsulated in the `createAction` function.

## Keep reading
**Next topic: [[Navigation]]**
