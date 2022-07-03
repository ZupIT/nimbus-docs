# forEach
Check the [specification](/specification/default-components/for-each.md) for the full definition of this structural component.

## Properties
```typescript
interface ForEachProps<T> {
  /**
   * The array of item to iterate over.
   */
  items?: T[],
  /**
   * The name of the implicit state containing the current item. You don't need to worry about this parameter unless you need to avoid shadowing in
   * nested forEach components. If in the inner most forEach you need to access the item of the outer most forEach, then their `iteratorName` must
   * have different values.
   * 
   * @defaultValue 'item'
   */
  iteratorName?: string,
  /**
   * The name of the implicit state containing the current index. You don't need to worry about this parameter unless you need to avoid shadowing in
   * nested forEach components. If in the inner most forEach you need to access the index of the outer most forEach, then their `indexName` must
   * have different values.
   * 
   * @defaultValue 'index'
   */
  indexName?: string,
  /**
   * Name of the property to identify each item in `items`. The index of the item is used when this is not specified. Specifying the key enhances
   * performance and avoid bugs, so use it whenever possible.
   */
  key?: string,
  /**
   * The template factory. This is a function that receives states referring to the current item and index as parameters. This function must
   * return the UI tree that `forEach` will use as a template.
   */
  children: (item: State<T>, index: State<number>) => JSX.Element | JSX.Element[],
}
```

## Example
```tsx
import { NimbusJSX, createState } from '@zup-it/nimbus-backend-core'
import { ForEach } from '@zup-it/nimbus-backend-core/components'
import { Column, Text } from '@zup-it/nimbus-backend-layout'
import { User } from '../model/user' // this interface is not important for this example

const users = createState<User[]>('users')

const MyScreen: Screen = () => (
  <Column state={users}>
    <Text>Here's all the people in the database:</Text>
    <Column>
      <ForEach items={users} key="id">
        {(item, index) => (
          <Text id="index">{index}</Text>
          <Text id="name">{item.get('name')}</Text>
          <Text id="age">{item.get('age')}</Text>
        )}
      </ForEach>
    </Column>
  </Column>
)
```


Suppose the state `users` is filled with the following array:

```typescript
[
  { "id": "u003", "name": "John", "age": 30 },
  { "id": "u009", "name": "Mary", "age": 22 },
  { "id": "u055", "name": "Anthony", "age": 5 },
]
```

Then, the previous code, renders the following UI tree:

```tsx
<Column state={users}>
  <Text>Here's all the people in the database:</Text>
  <Column>
    <Text id="index:u003">0</Text>
    <Text id="name:u003">John</Text>
    <Text id="age:u003">30</Text>
    <Text id="index:u009">1</Text>
    <Text id="name:u009">Mary</Text>
    <Text id="age:u009">22</Text>
    <Text id="index:u055">2</Text>
    <Text id="name:u055">Anthony</Text>
    <Text id="age:u055">5</Text>
  </Column>
</Column>
```
