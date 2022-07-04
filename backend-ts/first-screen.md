# Creating your first screen
Now that you installed and created your project with our cli tool, it's time to create a screen!

## 1. Create the new file
Under `src/screens`, create the file `my-screen.tsx` with the following content:
```tsx
import { NimbusJSX } from '@zup-it/nimbus-backend-core'
import { Text } from '@zup-it/nimbus-backend-layout'

export const MyScreen = () => <Text>Hello from my first screen!</Text>
```

> You can also use the CLI tool for creating this screen: `nimbus-ts generate-screen my-screen`. To know more about
this CLI command, check [this topic](cli.md#creating-screens-from-the template).

First, notice that the file extension for the screen is `tsx` and not `ts`. This is important because our screens are
written in TSX, which is an extended version of Typescript. It's very easy to identify TSX code, you just need to look
for tags that look like HTML, e.g. `<Column></Column>`. TSX code will always need the tsx extension and a special
import: the TSX interpreter, in our case `NimbusJSX`.

When reading about TSX, you'll see the terms JSX and TSX used intermittently. Both are correct and they work just like
JS and TS. JSX is the general term, while TSX is JSX extended to support typescript types.

JSX was introduced by React, which is a great tool for web development that inspired much of the today's architectures
for declarative UIs. For this reason, you'll see that writing for Nimbus using the backend TS is very similar to writing
React code. This is good because React is a very well known tool and might easy the learning curve for developers that
are new to Nimbus.

[comment]: <> (The previous example should use a Fragment and this paragraph should talk about it. Copy it from the beagle backend ts docs.)

To know more about JSX and TSX, please read the original documentation in
[the React's docs](https://reactjs.org/docs/jsx-in-depth.html). Just notice that instead of importing `React`,
as stated in their docs, here you're going to import `NimbusJSX`. It's also important to remember that HTML tags are
not accepted in Nimbus, even though they are accepted in React.

## 2. Create the new route
Edit the file `src/screens/index.ts` to add the new route:

```typescript
// ...
import { RouteMap } from '@zup-it/nimbus-backend-express'
import { myScreen } from './my-screen'

export const routes: RouteMap = {
  // ...
  '/my-screen': MyScreen,
}
```

In the code above we set up the screen we created before as the screen to be rendered when the URL "$baseUrl/my-screen"
is accessed. "$baseUrl" is the baseUrl given when creating the NimbusApp instance, at `src/index.ts`. If no baseUrl
was provided, then the route will be created at the servers root.

The routes, i.e, the keys of the map in the code above can be any valid express route. e.g. `'product/:id'` is a route
with a parameter: the id. To know more about creating express routes, read
[this article](http://expressjs.com/en/guide/routing.html).

The screens, i.e., the values of the map in the code above can be either the screen, directly, or an object containing
the screen and the method of the request (default is `get`). See the example below for a route that register a new
user:

```typescript
// ...
export const routes: RouteMap = {
  // ...
  '/user': { method: 'post', screen: NewUser },
}
```

Most of the times your screens will only be retrieving data and not modifying it, so we recommend not changing the
default behavior, which is `get` requests, unless you really need to and know what you're doing.

For some routes, we'll need to receive data from the request. e.g. if we use a route parameter, we need to get this
value somehow. If we create a `post` route, we'll need to get the body of the request. To retrieve this data, when
building the screen, we use the type `Screen`, which will be explained in the next documentation topic
([[Giving a type to your screens]]).

## 3. Run it!
If your project is already running, go to the browser and access `http://localhost:3000/my-screen`. If you have
changed the domain, port or baseUrl of the project generated with the CLI tool, use the address you set up.

If the project is not yet running, first run `yarn start` at the root of your project.

The browser should show the JSON corresponding to the screen you just created!

## Read next
:point_right: [Giving a type to your screens](screen-type.md)
