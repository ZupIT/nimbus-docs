# Getting started

## Contents
1. [Pre-installation](#pre-installation)
1. [Installation](#installation)
1. [Creating a new project](#creating-a-new-project)
1. [Running the application](#running-the-application)
1. [Full documentation for the CLI](#full-documentation-for-the-cli)
1. [Keep reading](#keep-reading)

## Pre-installation
1. Make sure you have Node installed by using the command `node -v`. The minimum node version compatible with this
lib is 13.2. If Node is not installed or is on an unsupported version, consider installing nvm in your machine. nvm is
a tool to manage node versions installed in the system, it's easier to upgrade and switch from one version to another
than a pure installation of node. To install nvm, check [this link](https://github.com/nvm-sh/nvm). If you don't want
to install nvm, you can also download node from [here](https://nodejs.org/en/).

2. This is optional, but we recommend using yarn instead of npm. If you choose to follow this recommendation, use the
command `npm install yarn --global`.

## Installation
In order to make it easier and faster to start up a project with Nimbus Backend TS, you should install our CLI tool
globally:

```shell
yarn global add @zup-it/nimbus-backend-cli
```

## Creating a new project
Using the CLI, run the command `new` with the name of your project:
```shell
nimbus-ts new [PROJECT NAME]
```

This command will create the following project structure:
```asc
📦 project-name
 ┣ 📂 src
 ┃ ┣ 📂 screens
 ┃ ┃ ┣ 📜 home.tsx  // example of navigation between screens
 ┃ ┃ ┣ 📜 index.ts  // file where the routes are defined
 ┃ ┃ ┗ 📜 welcome.tsx  // example for default page
 ┃ ┣ 📜 config.ts  // file that have methods to get the NimbusTsConfig
 ┃ ┣ 📜 global-context.ts  // declaration of GlobalContext and example for its usage
 ┃ ┗ 📜 index.ts  // root of the express app, where everything is set up
 ┣ 📜 .editorconfig
 ┣ 📜 .eslintrc.js
 ┣ 📜 LICENSE
 ┣ 📜 README.md
 ┣ 📜 nimbus-ts.json  // configuration file to be used by the CLI and the Express App  
 ┣ 📜 package-lock.json
 ┣ 📜 package.json
 ┗ 📜 tsconfig.json
```

## Running the application:
```shell
cd [PROJECT NAME]
yarn start
```

The application will be running at `http://localhost:3000` if you didn't change any of the default configurations.
This command also starts a websocket server that enables hot-reloading in frontend applications. To know more
about this, read this [topic](cli.md#starting-up-the-server-and-hot-reloading).

## Full documentation for the CLI
Read the topic [topic](cli.md) to know more about every command and options of the CLI.

## Read next
:point_right: [Create your first screen](first-screen.md)
