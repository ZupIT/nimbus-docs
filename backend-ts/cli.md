# CLI

## Contents
1. [Introduction](#introduction)
1. [Starting up the server and Hot Reloading](#starting-up-the-server-and-hot-reloading)
1. [Options for the command to start up a server](#options-for-the-command-to-start-up-a-server)
1. [Creating screens from the template](#creating-screens-from-the-template)
1. [Options for the command to create a project](#options-for-the-command-to-create-a-project)
1. [Options for the command to create a screen](#options-for-the-command-to-create-a-screen)
1. [Configuration file](#configuration-file)

## Introduction
The CLI for the Nimbus Backend TS is a tool to quickly create a new project and generate screens. To get started with it, please read the topic [Getting started](getting-started.md). This article aims at explaining details about every command and option available in the tool.

## Starting up the server and Hot Reloading
The CLI has a command to start up the server and restart it whenever the code base changes. This command also starts a websocket server that enables hot-reloading for front-end applications:

```
nimbus-ts start
```

You must run this at the root of your project and, by default, it will run the file `src/index.ts`. The hot-reloading server is also started at `ws://localhost:3001` by default. To change these options you can use the command line options explained [here](#options-for-the-command-to-start-up-a-server).

### Hot Reloading
Hot reloading is a common tool in frontend development where you alter the code and see the changes immediately in the web page or smartphone emulator. After running this command, every time your code is saved, it will send a message to every client connected to the websocket saying they should update. The frontend application will get this message and refresh the screen.

When a client connects, it should send a message to the server with a JSON containing the key "platform". e.g. `{"platform":"Flutter"}`. After that it must listen for messages containing the string "reload". This behavior is currently implemented in Nimbus Flutter and you just need to enable it by passing `enableHotReloading: true` to the Nimbus library. This will be soon be available out of the box for Web, Android and iOS.

## Options for the command to start up a server
```shell
nimbus-ts start [options]
```

**Options:**
| Option | Alias | Description | Default Value |
| ------ | ----- | ----------- | ------------- |
| --hot-reloading-disabled | -hrd | Disables the hot reloading websocket server. | `false` |
| --hot-reloading-port `<number>` | -hrp `<number>` | Changes the port for the hot reloading websocket server. | `3001` |
| --entrypoint `<string>` | -e `<string>` | Changes the application entrypoint. | `"src/index.ts"`

## Creating screens from the template
Go to the project root and run:
```
nimbus-ts generate-screen [acreen-name]
```

The new screen will be generated inside the folder: `src/screens`, or inside the folder defined on the attribute `screensFolderPath` in the configuration file `./nimbus-ts.json`.

The generated screen code will be:
```tsx
/** ./src/screens/screen-name.tsx */

import { NimbusJSX } from '@zup-it/nimbus-backend-core'
import { Text } from '@zup-it/nimbus-backend-layout'

export const ScreenNameScreen = () => (
  <>
    <Text>This the ScreenNameScreen screen component!</Text>
  </>
)
```

### Routing
When you generate a screen, the new screen is added to the `RouteMap` of the application (`src/screens/index.ts`). See below:
```tsx
import { RouteMap } from '@zup-it/nimbus-backend-express'
import { Home } from './home'
import { Welcome } from './welcome'
import { ScreenName } from './screen-name'

export const routes: RouteMap = {
  '': Welcome,
  '/home': Home,
  '/screen-name': ScreenName,
}
```

## Options for the command to create a project
```shell
nimbus-ts new [project-name] [options]
```

**Arguments:**
| Argument | Description |
| -------- | ----------- |
| [project-name] | Name of the project. eg. "name-of-my-project" (will be the name of the folder). |

**Options:**
| Option | Alias | Description | Default Value |
| ------ | ----- | ----------- | ------------- |
| --base-path `<string>` | -bp `<string>` | Base path that will be the root of the API. | `''` |
| --port `<number>` | -p `<number>` | Port where the service will run. | `3000` |

## Options for the command to create a screen
```shell
nimbus-ts generate-screen [screen-name] [options]
# or
nimbus-ts gs [screen-name] [options]
```

**Arguments:**
| Argument | Description |
| -------- | ----------- |
| [screen-name] | Name of the screen that will be generated. eg. "name-of-my-screen". |

**Options (all options are `boolean`):**
| Option | Alias | Description | Default Value |
| ------ | ----- | ----------- | ------------- |
| --with-route-params | -wrp | The screen will expect parameters in the url. | `false` |
| --with-headers | -wh | The screen will expect headers to in the request. | `false` |
| --with-body | -wb | The screen will expect the request to have a body. Invalid for "GET" requests. | `false` |
| --with-query | -wq | The screen will expect query parameters in the URL. | `false` |

> These options alter the code generated for the screen so it comes with all the boilerplate necessary for implementing the requested features.

#### Example
```
nimbus-ts generate-screen screen-name 
```
or:
```
nimbus-ts gs screen-name 
```

The new screen will be generated according to the given options, so you can write your code faster. The screen generated in this example will be:
```tsx
/** ./src/screens/screen-name.tsx */

import { NimbusJSX } from '@zup-it/nimbus-backend-core'
import { Text } from '@zup-it/nimbus-backend-layout'
import { Screen } from '@zup-it/nimbus-backend-express'

export const ScreenName: Screen = () => (
  <>
    <Text>This the ScreenName screen component!</Text>
  </>
)
```

## Configuration file
When you create a new project with the CLI, besides the whole boilerplate structure, a new configuration file is created at the root of the new project: `nimbus-ts.json`. This configuration file will have some attributes required by the CLI to generate screens and also start the express server.

> Note: If you want to handle the express attributes with your own strategy, you don't need the configuration file on your application, but you MUST HAVE it if you want to use the generate screen command.

The configuration file will have the following structure:
```json
{
  "projectName": "first-one",
  "port": "3000",
  "basePath": "",
  "screensFolderPath": "src/screens",
  "routes": {
    "filePath": "src/screens/index.ts",
    "varName": "routes"
  }
}
```

### Attributes
Documentation of each attribute of the configuration file.

| Attribute | Description | Default Value |
| --------- | ----------- | ------------- |
| `projectName` | The name of your project | `"" (empty string)` |
| `port ` | The port where the express app will be running. | `3000` |
| `basePath` | The base path of your exposed service. eg: if is set as "api", your service will be like this: "https://localhost:3000/api/[your-route] | `"" (empty string)` |
| `screensFolderPath` | The path to your screens files folder. | `src/screens` |
| `routes.filePath` | Path to the file where the routes are defined. | `src/screens/index.ts` |
| `routes.varName` | The name of the variable where the routes are defined inside the file defined on the `filePath` property. | `routes` |

