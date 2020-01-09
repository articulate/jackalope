<p align="center">
  <a href="#"><img src="https://cloud.githubusercontent.com/assets/888052/22172037/543e7f10-df6b-11e6-8ab8-8a1e679dd27e.png" alt="paperplane" style="max-width:100%;"></a>
</p>
<p align="center">
  Lighter-than-air node.js server framework.
</p>
<p align="center">
  <a href="https://www.npmjs.com/package/paperplane"><img src="https://img.shields.io/npm/v/paperplane.svg" alt="npm version" style="max-width:100%;"></a> <a href="https://www.npmjs.com/package/paperplane"><img src="https://img.shields.io/npm/dm/paperplane.svg" alt="npm downloads" style="max-width:100%;"></a> <a href="https://travis-ci.org/articulate/paperplane"><img src="https://travis-ci.org/articulate/paperplane.svg?branch=master" alt="Build Status" style="max-width:100%;"></a> <a href='https://coveralls.io/github/articulate/paperplane?branch=v2'><img src='https://coveralls.io/repos/github/articulate/paperplane/badge.svg?branch=v2' alt='Coverage Status' /></a>
</p>

## Documentation

- [Getting started guide](https://github.com/articulate/paperplane/blob/master/docs/getting-started.md)
- [API docs](https://github.com/articulate/paperplane/blob/master/docs/API.md)
- [Migration guide](https://github.com/articulate/paperplane/blob/master/docs/migration-guide.md)

## Introduction

The main goal of `paperplane` is to make building a `node.js` server easy, without all of the configuration or imperative boilerplate required for other server frameworks.  If you prefer to build apps with function composition or even a point-free style, then `paperplane` is for you.

With `paperplane` you get all of these out-of-the-box:

- pure, functional, Promise-based [request handlers](https://github.com/articulate/paperplane/blob/master/docs/getting-started.md#basic-concepts)
- support for request handlers that return [Algebraic Data Types](https://github.com/articulate/paperplane/blob/master/docs/getting-started.md#what-are-algebraic-data-types) - **new in v2.0**
- support for highly scalable [serverless deployment](https://github.com/articulate/paperplane/blob/master/docs/API.md#serverless-deployment) - **new in v2.1**
- composeable json [body parsing](https://github.com/articulate/paperplane/blob/master/docs/API.md#parsejson)
- dead-simple [routing functions](https://github.com/articulate/paperplane/blob/master/docs/API.md#routes)
- several common [response helpers](https://github.com/articulate/paperplane/blob/master/docs/getting-started.md#response-object)
- json-formatted [logging](https://github.com/articulate/paperplane/blob/master/docs/API.md#logger)
- easily configurable [CORS support](https://github.com/articulate/paperplane/blob/master/docs/API.md#cors)
- plug-n-play [static file serving](https://github.com/articulate/paperplane/blob/master/docs/API.md#serve)

Let's try a quick Hello World example server.  It accepts a `:name` param in the url, and then includes that name in the `json` response body.

```js
const { compose } = require('ramda')
const http = require('http')
const { json, logger, methods, mount, routes } = require('paperplane')

const hello = req => ({
  message: `Hello ${req.params.name}!`
})

const app = routes({
  '/hello/:name': methods({
    GET: compose(json, hello)
  })
})

http.createServer(mount({ app })).listen(3000, logger)
```

So simple and functional, with an easily readable routing table and pure functions for the route handler.  If that sounds like fun to you, then read the [Getting started guide](https://github.com/articulate/paperplane/blob/master/docs/getting-started.md) or the [API docs](https://github.com/articulate/paperplane/blob/master/docs/API.md) to learn more.

## Example application

To help you learn the concepts used in paperplane, check out the [demo application](https://github.com/articulate/paperplane/blob/master/demo).

If you have docker installed, you can run the demo locally:

1. Clone this repo
1. If you're using Docker Desktop for Windows:
    - `cp docker-compose.override.windows.yml docker-compose.override.yml`
1. `docker-compose up`
1. http://localhost:3000
