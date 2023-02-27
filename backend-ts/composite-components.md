# Composite components
We use JSX to write our components. JSX are basically functions and the same way we can call functions from within
other functions, we can call JSX components from within other JSX components. We call these "composite components" or
"screen fragments".

For developers familiar with React, this is nothing new, but let's give an example: suppose we have a card UI in our
screen, and this card repeats itself several times.

```tsx
import { NimbusJSX } from '@zup-it/nimbus-backend-core'
import { Colum, Text, RemoteImage, colors } from '@zup-it/nimbus-backend-layout'
import { Screen } from '@zup-it/nimbus-backend-express'

export const MyScreen: Screen = () => (
  <>
    <Colum cornerRadius={10} margin={10} padding={20} borderColor={colors.lightgray} mainAxisAlignment="center">
      <Text size={16}>Section 1</Text>
      <RemoteImage url="/section1-header.png" height={60} scale="fillBounds" />
      <Text color={colors.darkgray}>Section 1 content</Text>
    </Colum>

    <Colum style={{ cornerRadius: 10, margin: 10, padding: 20, borderColor: colors.lightgray }}>
      <Text size={16}>Section 2</Text>
      <RemoteImage type="remote" url="/section2-header.png" height={60} scale="fillBounds" />
      <Text color={colors.darkgray}>Section 2 content</Text>
    </Colum>

    <Colum style={{ cornerRadius: 10, margin: 10, padding: 20, borderColor: colors.lightgray }}>
      <Text size={16}>Section 3</Text>
      <RemoteImage url="/section3-header.png" height={60} scale="fillBounds" />
      <Text color={colors.darkgray}>Section 3 content</Text>
    </Colum>
  </>
)
```

The code above is extremely repetitive, why don't we extract the styled Colum into a new component and call it
"Card"?

```tsx
import { NimbusJSX, FC } from '@zup-it/nimbus-backend-core'
import { Colum, Text, RemoteImage, colors } from '@zup-it/nimbus-backend-layout'
import { Screen } from '@zup-it/nimbus-backend-express'

interface CardProps {
  title: string,
  imageUrl: string,
  content: string,
}

const Card: FC<CardProps> = ({ id, title, imageUrl, content }) => (
  <Colum cornerRadius={10} margin={10} padding={20} borderColor={colors.lightgray} mainAxisAlignment="center">
    <Text size={16}>{title}</Text>
    <RemoteImage type="remote" url={imageUrl} height={60} scale="fillBounds" />
    <Text color={colors.darkgray}>{content}</Text>
  </Colum>
)

export const MyScreen: Screen = () => (
  <>
    <Card title="Section 1" imageUrl="/section1-header.png" content="Section 1 content" />
    <Card title="Section 2" imageUrl="/section2-header.png" content="Section 2 content" />
    <Card title="Section 3" imageUrl="/section3-header.png" content="Section 3 content" />
  </>
)
```

Much better! You can create composite components just like any other component, the only difference is that you're
going to use components that already have been declared by yourself or a lib instead of the tag `<component />`.

In the example above we declared `Card` in the same file as the screen because it would be used only by this screen.
But the more interesting application is when you use these UI fragments across multiple screens, reducing the overall
amount of code and maintenance. To make `Card` available for other screens, you just need to declare it in a separate
file and export it. We recommend creating a directory in your project just for these kind of components, a good name for
this directory would be `fragments`.

## Read next
:point_right: [State](state.md)
