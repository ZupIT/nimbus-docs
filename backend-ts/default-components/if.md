# if, then, else
Check the [specification](/specification/default-components/if.md) for the full definition of this structural component.

## Properties
```typescript
interface IfProps {
  /**
   * If true, shows the contents of `Then`. Otherwise, if `Else` exists, shows the contents of `Else`, otherwise, shows nothing.
   */
  condition: Expression<boolean>,
  /**
   * If should always be accompanied with Then or Then and Else.
   */
  children: JSX.Element | [JSX.Element, JSX.Element],
}
```

`Then` and `Else`have no properties other than `children`, which in both cases has the type `JSX.Element | JSX.Element[]`.

## Example
```tsx
import { NimbusJSX, createState } from '@zup-it/nimbus-backend-core'
import { ForEach } from '@zup-it/nimbus-backend-core/components'
import { Column, Text, LocalImage } from '@zup-it/nimbus-backend-layout'

const isMorning = createState<boolean>('isMorning')

const MyScreen: Screen = () => (
  <Column state={isMorning}>
    <If condition={isMorning}>
      <Then>
        <Text>Good morning!</Text>
        <LocalImage id="sun" />
      </Then>
      <Else>
        <Text>Good evening!</Text>
        <LocalImage id="moon" />
      </Else>
    </If>
  </Column>
)
```

Suppose we set the value of `isMorning` to `true`. Then, the rendered tree is the following:
```tsx
<Column state={isMorning}>
  <Text>Good morning!</Text>
  <LocalImage id="sun" />
</Column>
```

Now suppose we set the value of `isMorning` to `false`.  Then, the rendered tree is the following:

```tsx
<Column state={isMorning}>
  <Text>Good evening!</Text>
  <LocalImage id="moon" />
</Column>
```
