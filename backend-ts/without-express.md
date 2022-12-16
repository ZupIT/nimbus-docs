# Using this lib without Express
If you need to use the backend for Nimbus in Typescript, but can't use Express, you should use only the libs
`@zup-it/nimbus-backend-core` and `@zup-it/nimbus-backend-components`. Everything in this documentation, but the
resources imported from `@zup-it/nimbus-backend-express` are valid for this scenario.

The main difference is that you won't have access to the type `Screen` and the dependencies injected into it, but
although they help a lot and prevent small mistakes, they're not necessary to develop a Nimbus Application. Instead of
using the type `Screen`, you should use `FC` to create the screens.

You'll also not have access to the `NimbusApp` class, so every route must be registered manually in the server
technology of your preference. The important thing here that has not been mentioned in the rest of this documentation is
the method [`serialize`](https://zupit.github.io/nimbus-backend-ts/modules/_zup_it_nimbus_backend_core.html#serialize).
This method is implicitly called by the `NimbusApp` when using `Express`, but in this case, it should be called manually.
This method is responsible for taking a JSX element (or an instance of Component) and serializing it into a JSON string.
See the example below:

```tsx
import { NimbusJSX, serialize } from '@zup-it/nimbus-backend-core'

const MyScreen: FC = () => <>Hello World</>

const myScreenAsJson: string = serialize(MyScreen())
```
