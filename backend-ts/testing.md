# Testing

## Contents
1. [Introduction](#introduction)
1. [Snapshots with Super Agent](#snapshots-with-super-agent)
1. [Keep reading](#keep-reading)

## Introduction
It is always important to test every aspect of your application and testing a Node project with Nimbus is just like
testing any Typescript project, there's no secret to it. As the testing tool, we recommend using
[Jest](https://jestjs.io/).

The CLI for the Nimbus Backend Typescript creates a project already containing all the dependencies you need for
testing and an example. This is the easiest way of setting up your project, but if you didn't use it, we recommend
reading the article
[Testing with Jest in TypeScript and Node.js for Beginners](https://javascript.plainenglish.io/beginners-guide-to-testing-jest-with-node-typescript-1f46a1b87dad).

If you're new to javascript testing, a very good article teaching basic technics for testing with Jest can be found
[here](https://blog.logrocket.com/testing-typescript-apps-using-jest/).

## Snapshots with Super Agent
If you feel you don't need unitary tests, we recommend that you create at least snapshot tests for every screen of
your application. They're super easy to create and can immediately detect basic errors in your code.

To create these tests we need to simulate requests to the Node server that will, in turn, create the screens. We then
need to compare the JSON we obtained with the JSON we have from the previous executions (snapshots).

To simulate the requests, we use the tool [Super Agent](https://www.npmjs.com/package/superagent). See the example
below for creating snapshot tests for the screens at `/welcome` and `/product`.

```typescript
import { agent, Response } from 'supertest'
import { expressApp, expressListener } from '../src'

function makeRequest(route: string) {
  return new Promise<Response>((resolve, reject) => {
    agent(expressApp)
      .get(route)
      .set('Accept', 'application/json')
      .expect('Content-Type', 'application/json; charset=utf-8')
      .expect(200)
      .end((err, resp) => {
        if (err) reject(err)
        else resolve(resp)
      })
  })
}

async function expectRouteToMatchSnapshot(route: string) {
  const resp = await makeRequest(route)
  expect(resp.body).toMatchSnapshot()
}

describe('Verifies if the screens outputs match the snapshots', () => {
  afterAll(() => expressListener.close())

  it('should render the welcome screen', () => expectRouteToMatchSnapshot('/welcome'))

  it('should render the product screen', () => expectRouteToMatchSnapshot('/product/1'))
})
```

In the code above, `expressApp` and `expressListener` are exported by the entrypoint of our application: `src/index.ts`.
See the example below:

```typescript
import express from 'express'
import cors from 'cors'
import { NimbusApp } from '@zup-it/nimbus-backend-express'
import { routes as nimbusRoutes } from './nimbus/screens'

const port = 3000
export const expressApp = express()

expressApp.use(cors()).use(express.json())

export const expressListener = expressApp.listen(port, () => {
  console.log(`App listening at http://localhost:${port}`)
})

new NimbusApp(expressApp, nimbusRoutes)
```
