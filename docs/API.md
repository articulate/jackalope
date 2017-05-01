# API

- [`cors`](#cors)           - CORS support wrapper
- [`html`](#html)           - response helper, type `text/html`
- [`json`](#json)           - response helper, type `application/json`
- [`logger`](#logger)       - json request logger
- [`methods`](#methods)     - maps request methods to handler functions
- [`mount`](#mount)         - top-level server function wrapper
- [`parseJson`](#parsejson) - json body parser
- [`redirect`](#redirect)   - redirect response helper
- [`routes`](#routes)       - maps express-style route patterns to handler functions
- [`send`](#send)           - basic response helper
- [`static`](#static)       - static file serving handler

### cors

```haskell
((Request -> (Response | Promise Response)), Object) -> Request -> Promise Response
```

Wraps a top-level handler function to add support for [CORS](http://devdocs.io/http/access_control_cors).  Lifts the handler into a `Promise` chain, so the handler can respond with either a [`Response`](https://github.com/articulate/paperplane/blob/master/docs/getting-started.md#response-object), or a `Promise` that resolves with one.  Also accepts an object with the following optional properties to override the default CORS behavior.

| Property | Type | Overridden header | Default |
| -------- | ---- | ----------------- | ------- |
| `credentials` | `String` | `access-control-allow-credentials` | `true` |
| `headers` | `String` | `access-control-allow-headers` | `content-type` |
| `methods` | `String` | `access-control-allow-methods` | `GET,POST,OPTIONS,PUT,PATCH,DELETE` |
| `origin`  | `String` | `access-control-allow-origin` | `*` |

```js
const { always } = require('ramda')
const http = require('http')
const { cors, mount, send } = require('paperplane')

const endpoint = req =>
  Promise.resolve(req.body).then(send)

const opts = {
  headers: 'x-custom-header',
  methods: 'GET,PUT'
}

const app = cors(endpoint, opts)

http.createServer(mount(app)).listen(3000)
```

### html

```haskell
(String | Buffer | Stream) -> Response
```

Returns a [`Response`](https://github.com/articulate/paperplane/blob/master/docs/getting-started.md#response-object), with the `content-type` header set to `text/html`.

```js
const { html } = require('paperplane')
const template = require('../views/template.pug')

const usersPage = () =>
  fetchUsers()
    .then(template)
    .then(html)
```

In the example above, it resolves with a [`Response`](https://github.com/articulate/paperplane/blob/master/docs/getting-started.md#response-object) similar to:

```js
{
  body: '<html>...</html>',
  headers: {
    'content-type': 'text/html'
  },
  statusCode: 200
}
```

### json

```haskell
Object -> Response
```

Returns a [`Response`](https://github.com/articulate/paperplane/blob/master/docs/getting-started.md#response-object), with a `body` encoded with `JSON.stringify`, and the `content-type` header set to `application/json`.

```js
const { json } = require('paperplane')

const users = () =>
  fetchUsers()
    .then(json)
```

In the example above, it resolves with a [`Response`](https://github.com/articulate/paperplane/blob/master/docs/getting-started.md#response-object) similar to:

```js
{
  body: '[{"id":1,"name":"Scott"}]',
  headers: {
    'content-type': 'application/json'
  },
  statusCode: 200
}
```

### logger

```haskell
Object -> ()
```

Logs request/response as `json`.  Uses the following whitelists:

- req: `['headers', 'method', 'url']`
- res: `['statusCode']`

Provided as an example logger to use with [`mount`](#mount), as below.

```js
const http = require('http')
const { logger, mount, send } = require('paperplane')

const app = () =>
  send() // 200 OK

http.createServer(mount(app, { logger })).listen(3000)
```

Logs will be formatted as `json`, similar to below:

```json
{"req":{"headers":{"host":"localhost:3000","connection":"keep-alive","cache-control":"no-cache","user-agent":"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.98 Safari/537.36","content-type":"application/json","accept":"*/*","accept-encoding":"gzip, deflate, sdch, br","accept-language":"en-US,en;q=0.8"},"method":"GET","url":"/courses/"},"res":{"statusCode":200}}
```

### methods

```haskell
{ k: (Request -> (Response | Promise Response)) } -> (Request -> Promise Response)
```

Maps handler functions to request methods.  Returns a handler function.  If the method of an incoming request doesn't match, it rejects with a `404 Not Found`.  Use in combination with [`routes`](#routes) to build a routing table of any complexity.

**Note:** If you supply a `GET` handler, `paperplane` will also use it to handle `HEAD` requests.

```js
const http = require('http')
const { methods, mount } = require('paperplane')

const { createUser, fetchUsers } = require('./api/users')

const app = methods({
  GET:  fetchUsers,
  POST: createUser
})

http.createServer(mount(app)).listen(3000)
```

### mount

```haskell
((Request -> (Response | Promise Response)), Object) -> (IncomingMessage, ServerResponse) -> ()
```

Wraps a top-level handler function to prepare for mounting as a new `http` server.  Lifts the handler into a `Promise` chain, so the handler can respond with either a [`Response`](https://github.com/articulate/paperplane/blob/master/docs/getting-started.md#response-object), or a `Promise` that resolves with one.  Also accepts an options object with `errLogger` and `logger` properties, both of which can be set to [`logger`](#logger).

**Note:**  The `errLogger` option is primarily intended for logging, not notifying your error aggregation service.  For that you will need to wrap your top-level handler function with [`paperplane-airbrake`](https://github.com/articulate/paperplane-airbrake) or something similar.

```js
const http = require('http')
const { logger, mount, send } = require('paperplane')

const app = req =>
  Promise.resolve(req.body).then(send)

const opts = { errLogger: logger, logger }

http.createServer(mount(app, opts)).listen(3000)
```

### parseJson

```haskell
Request -> Request
```

Parses the request body as `json` if available, and if the `content-type` is `application/json`.  Otherwise, passes the [`Request`](https://github.com/articulate/paperplane/blob/master/docs/getting-started.md#request-object) through untouched.

```js
const { compose } = require('ramda')
const http = require('http')
const { mount, parseJson, json } = require('paperplane')

const echo = req =>
  Promise.resolve(req.body).then(json)

const app = compose(echo, parseJson)

http.createServer(mount(app)).listen(3000)
```

### redirect

```haskell
(String, Number) -> Response
```

Accept a `Location` and optional `statusCode` (defaults to `302`), and returns a [`Response`](https://github.com/articulate/paperplane/blob/master/docs/getting-started.md#response-object) denoting a redirect.

**Pro-tip:** if you want an earlier function in your composed application to respond with a redirect and skip everything else, just wrap it in a `Promise.reject` (see example below).  The error-handling code in `paperplane` will ignore it since it's not a real error.

```js
const { compose, composeP } = require('ramda')
const http = require('http')
const { html, methods, mount, parseJson, routes, send } = require('paperplane')

const login = require('./views/login')

// Please make your authorization better than this
const authorize = req =>
  req.headers.authorization
    ? Promise.resolve(req)
    : Promise.reject(redirect('/login'))

const echo = req =>
  Promise.resolve(req.body).then(send)

const app = routes({
  '/echo': methods({
    POST: composeP(echo, authorize)
  }),

  '/login': methods({
    GET: compose(html, loginPage)
  })
})

http.createServer(mount(app)).listen(3000)
```

In the example above, `redirect()` returns a [`Response`](https://github.com/articulate/paperplane/blob/master/docs/getting-started.md#response-object) similar to:

```js
{
  body: '',
  headers: {
    Location: '/login'
  },
  statusCode: 302
}
```

### routes

```haskell
{ k: (Request -> (Response | Promise Response)) } -> (Request -> Response)
```

Maps handler functions to express-style route patterns.  Returns a handler function.  If the path of an incoming request doesn't match, it rejects with a `404 Not Found`.  Use in combination with [`methods`](#methods) to build a routing table of any complexity.

```js
const http = require('http')
const { mount, routes } = require('paperplane')

const { fetchUser, fetchUsers, updateUser } = require('./lib/users')

const app = routes({
  '/users': methods({
    GET: fetchUsers
  }),

  '/users/:id': methods({
    GET: fetchUser,
    PUT: updateUser
  })
})

http.createServer(mount(app)).listen(3000)
```

### send

```haskell
(String | Buffer | Stream) -> Response
```

The most basic response helper.  Simply accepts a `body`, and returns a properly formatted [`Response`](https://github.com/articulate/paperplane/blob/master/docs/getting-started.md#response-object), without making any further assumptions.

```js
const { send } = require('paperplane')

send('This is the response body')
```

In the example above, it returns a [`Response`](https://github.com/articulate/paperplane/blob/master/docs/getting-started.md#response-object) similar to:

```js
{
  body: 'This is the response body',
  headers: {},
  statusCode: 200
}
```

### static

```haskell
Object -> (Request -> Response)
```

Accepts an options object (see [details here](https://www.npmjs.com/package/send#options)), and returns a handler function for serving static files.  Expects a `req.params.path` to be present on the [`Request`](https://github.com/articulate/paperplane/blob/master/docs/getting-started.md#request-object), so you'll need to format your route pattern similar to the example below, making sure to include a `/:path+` route segment.

```js
const http = require('http')
const { mount, routes, static } = require('paperplane')

const app = routes({
  '/public/:path+': static({ root: 'public' })
})

http.createServer(mount(app)).listen(3000)
```
