# Operations
[Nimbus States](/state.md) are evaluated when run by the frontend, so we can't use pure
javascript operators or functions to manipulate them. For example, if `results` is a State, we can't do
`results.get('page') + 1` to calculate the next page; we can't do `results.get('data').length` to know the number
of elements in the result table. `results.get('page')` is of type `StateNode<number>`, it's not the primitive type
`number`; and `results.get('data')` is of type `StateNode<any[]>`, which is not an array. They both are only
references to values that will exist in the frontend at runtime, since we don't know their values when creating the
screen in the backend, we can't treat them as common JS variables, we don't know their values.

Applying an operation to a state value is very important though. If we can't use JS operators or common functions,
we must be able to do it in some other form. This is what the Nimbus Operations are for! They're a way of manipulating
the Nimbus state.

Check the [specification](/specification/operation.md) to know more about the definition of Nimbus Operations.

# Using Operations
Operations in Nimbus are instances of the class `Operation<T>`, where `T` is the return type of the Operation. But you
don't really need to know this for using them, their use is very simple and intuitive, you can use them just like any
JS function. See the example below:

```tsx
import { NimbusJSX, createState } from '@zup-it/nimbus-backend-core'
import { sum } from '@zup-it/nimbus-backend-core/operations'
import { Screen } from '@zup-it/nimbus-backend-express'
import { Button } from './components/button' // the implementation of this component is not important for this example

const counter = createState('counter', 0)

export const MyScreen: Screen = <Button onPress={counter.set(sum(counter, 1))}>value: {counter}</Button>
```

Above, `sum` is a Nimbus Operation factory, but we can call it just an operation.

Since it's much easier to deal with Nimbus Operations when they are functions, instead of instantiating `Operation` by
ourselves, we use functions that do this for us. The code becomes much more intuitive. The syntax is just like any other
JS function and the fact that it returns an instance of `Operation<number>` is hidden from the developer, being used
only for type checking by the compiler.

# Creating your own Operations
Nimbus already comes with some [pre-defined operations](/specification/default-operations.md), but for most applications, they're not enough and we
need to extend the set of operations.

### First step
The first step when creating a custom operation is deciding on its signature. We must know what the name is going to
be and what arguments it will accept. It is important to remark here that Operation names must be unique, Nimbus does
not support "operation overloading".

Let's say we want an Operation to format phone numbers: "formatPhoneNumber(number: string, country: string) => string".
The name of the operation is "formatPhoneNumber" and it accepts 2 arguments, the phone number itself and a country
acronym, both strings. The Operation, after executed produces a value of type string.

### Second step
Now that we know the Operation we need to implement, let's declare it in the Backend TS:

```typescript
import { Operation } from '@zup-it/nimbus-backend-core'

/**
 * Formats a phone number.
 *
 * @param number the phone number to format.
 * @param country a country acronym so the format can be correctly localized.
 * @returns an instance of Operation<string>. i.e. an operation that results in a string when run by the frontend.
 */
export const formatPhoneNumber = (
  number: Expression<string>,
  country: Expression<string>,
) => new Operation<string>('formatPhoneNumber', [number, country])
```

Above, we created a Nimbus Operation Factory, it creates operations of the type "formatPhoneNumber". It's a function
and it receives a phone number (first parameter) and a country acronym (second parameter). The Operation returned is
of type `Operation<string>`, because when run in the frontend, this operation yields a string. To create the Operation
we called the class constructor, which receives the operation name as the first parameter (must be the same as the one
declared in the frontend) and an array of arguments as the second parameter.

Notice that both arguments have been typed as Expressions, this means the function will accept any string, but also
States and Operations that represent strings.

Before the function itself we wrote a special type of comment. This is called JSDocs and is very important for making
it easier to use and maintain the code. We advise using it whenever it's possible.

### Third step
The third step is to actually implement the operation in the frontend, which is out of the scope of this text. Please
read the [documentation](/) corresponding to the frontend lib you're using to learn exactly how
to do this.

# Read next
:point_right: Actions](/action.md)
