# Expressions
A big problem with [Nimbus expressions](/specification/expression.md) is that they are strings and therefore can't be typed.

**Example:**
Suppose a button that receives the attribute `enabled`, which tells if it's clickable or not. This attribute must be a boolean. Since we
can we use expressions, it must also accept strings and we can't know if the string is an expression or if the expression results in a Boolean.

We can't solve this problem in pure JSON, but we can do something about it in Typescript. Instead of dealing with strings, the Nimbus Backend TS
uses State objects and Operation objects, both are converted to expression strings when serialized. This way, we can always type our data, even when
they're supposed to accept expressions.

To type something as `T` or an expression that results in `T`, use the type `Expression<T>`. If the type can only be an expression and not a literal,
use `DynamicExpression<T>`. Both types can be imported from `@zup-it/nimbus-backend-core`.

In summary, you don't need to worry about expression strings in Nimbus Backend TS. By using [States](state.md) and [Operations](operation.md) you're
already implicitly using expressions. The lib will take care of all the serialization into the actual strings expected by the frontend.

## Reading values from a state or operation results
To read from `Expression<T>` you should use the methods `get` and `at`.

- `get` access properties of an expression that refers to an object;
- `at` access indexes of an expression that refers to an array.

See the example below:

```tsx
import { NimbusJSX, createState } from '@zup-it/nimbus-backend-core'
import { Column, Text } from '@zup-it/nimbus-backend-layout'
import { Screen } from '@zup-it/nimbus-backend-express'

interface Person {
  name: string,
  age: number,
  nicknames: string[],
}

const person = createState<Person>('person', {
  name: 'John',
  age: 30,
  nicknames: ['J', 'Dude', 'Guy']
})

export const MyScreen = () => (
  <Column state={person}>
    <Text>
      Hello {person.get('name')}! You're {person.get('age')} years old and also attends by the names
      {person.get('nicknames').at(0)}, {person.get('nicknames').at(1)} and {person.get('nicknames').at(2)}.
    </Text>
  </Column>
)
```

Above, we use `get` when the state is an object and `at` when it's an array. But don't worry too much, TS will take
care of this for you, it will only accept `get` when it's an object, only `at` when it's an array and when it's a
primitive type, it won't accept neither `get` nor `at`.

Tip: normally, to display the contents of an array, instead of accessing its indexes, we want to iterate over all of its
items. To do so, you'd need the structural component [`forEach`](default-components/for-each.md). You could also format
an array into a string, for this, you'd need a [custom operation](operation.md#creating-your-own-operations).
