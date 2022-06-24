A big problem with expressions is that they are string and therefore can't be typed.

Suppose a button that receives the attribute "enabled" to know if the button should enabled or disabled. This attribute must be a boolean. Since we
can we use expressions, it must also accept strings and we can't
know if the string is an expression or if the expression results in a Boolean.

---

## Reading from a Context
To read from a Beagle Context you should use the methods `get` and `at`.

- `get` access properties of a context that refers to an object;
- `at` access indexes of a context that refers to an array.

See the example below:

```tsx
import { BeagleJSX, createContext } from '@zup-it/beagle-backend-core'
import { Container } from '@zup-it/beagle-backend-components'
import { Screen } from '@zup-it/beagle-backend-express'

interface Person {
  name: string,
  age: number,
  nicknames: string[],
}

const person = createContext<Person>('person', {
  name: 'John',
  age: 30,
  nicknames: ['J', 'Dude', 'Guy']
})

export const MyScreen = () => (
  <Container context={person}>
    <>
      Hello {person.get('name')}! You're {person.get('age')} years old and also attends by the names
      {person.get('nicknames').at(0)}, {person.get('nicknames').at(1)} and {person.get('nicknames').at(2)}.
    </>
  </Container>
)
```

Above, we use `get` when the context is an object and `at` when it's an array. But don't worry too much, TS will take
care of this for us, it will only accept `get` when it's an object, only `at` when it's an array and when it's a
primitive type, it won't accept neither `get` nor `at`.

Tip: normally, to display the contents of an array, instead of accessing its indexes, we want to iterate over all of its
items. To do so, you'd need a
[ListView](https://zupit.github.io/beagle-backend-ts/modules/_zup_it_beagle_backend_components.html#ListView) or a
[GridView](https://zupit.github.io/beagle-backend-ts/modules/_zup_it_beagle_backend_components.html#GridView). To format
an array into a string, you'd need a [[custom operation|Operations#creating-your-own-operations]].

## Writing to a Context
To modify the content of a Context, we need an [[Action|Actions]]. To create this action, we can call the method `set`
of the context. See the example below:

```tsx
import { BeagleJSX, createContext } from '@zup-it/beagle-backend-core'
import { sum } from '@zup-it/beagle-backend-core/operations'
import { Button } from '@zup-it/beagle-backend-components'
import { Screen } from '@zup-it/beagle-backend-express'

const counter = createContext('counter', 0)

export const MyScreen: Screen = <Button onPress={counter.set(sum(counter, 1))}>value: {counter}</Button>
```

Above, we created a screen with a single button where the label increments by one every time it's clicked. `context.set`
will only accept values of its type or Expressions of it. `sum` is one of the [[Operations]] shipped with
Beagle.

> Global Contexts can be set outside Beagle by the frontend application. To know more check the
[documentation](https://docs.usebeagle.io) for the frontend lib you're using.