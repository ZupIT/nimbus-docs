# Actions
Check the [specification](/specification/action.md) to know more about the definition of Nimbus Actions. 

## Using Actions
Nimbus Actions are instances of the class `Action`, but just like [`Operations`](operation.md), instead of instantiating the class ourselves, we use
Action factories (functions). This makes the code more readable and easy to use. See the example below:

```tsx
import { NimbusJSX } from '@zup-it/nimbus-backend-core'
import { log } from '@zup-it/nimbus-backend-core/actions'
import { Button } from '@zup-it/nimbus-backend-components'
import { Screen } from '@zup-it/nimbus-backend-express'

const MyScreen: Screen = () => <Button onPress={log('Hi!')}>Click me!</Button>
```

The code above renders a button that when clicked logs the message "Hi!".

## Default Actions
Nimbus ships with some important actions already. Below you can find a list with all of them and links to their
respective documentation.

- [Navigation actions](default-actions/navigation.md);
- [`condition`](default-actions/condition.md);
- [`log`](default-actions/log.md);
- [`openUrl`](default-actions/open-url.md);
- [`sendRequest`](default-actions/send-request.md);
- [`setContent`](default-actions/set-content.md);
- [`setState` (`state.set`)](default-actions/set-state.md).

## Creating your own actions
It's very simple to create custom Actions. The examples below show how to create an action called "notify". The idea
is that whenever "notify" is called, a floating message appears in the screen and disappears after a while.

```typescript
import { Expression, Actions } from '@zup-it/nimbus-backend-core'
import { Action } from '@zup-it/nimbus-backend-core/model/action' // not recommended, we'll see another approach later

interface Notify {
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
export function notify({ message, title, icon, onPress }: Notify) {
  return new Action({
    name: 'notify',
    properties: { message, title, icon, onPress },
  })
}
```

First, notice that we make a strange import of the class `Action`. This is not recommended, but it's useful for this
example, in the next we'll show a better way of achieving the same result.

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

This second example is much more simple than the first and does the exact same thing. The only difference is that most
of the code of the first example is now encapsulated in the `createAction` function.

# Read next
:point_right: [Expressions](expression.md)
