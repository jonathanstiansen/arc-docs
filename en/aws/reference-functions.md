# Authoring cloud functions

## `@architect/functions` is for creating cloud function signatures

- [`arc.html.get(...fns)`](#arc.html.get)
- [`arc.html.post(...fns)`](#arc.html.post)
- [`arc.json.get(...fns)`](#arc.json.get)
- [`arc.json.post(...fns)`](#arc.json.post)
- [`arc.events.subscribe((payload, callback)=>)`](#arc.events.subscribe)
- [`arc.events.publish(params, callback)` where `params` requires `name` and `payload` and optionally `app` keys](#arc.events.publish)
- [`arc.tables.insert((record, callback)=>)`](#arc.tables.insert)
- [`arc.tables.update((record, callback)=>)`](#arc.tables.update)
- [`arc.tables.destroy((record, callback)=>)`](#arc.tables.destroy)

`arc.html` and `arc.json` functions accept one or more Express-style middleware functions `(req, res, next)=>`.

---

## <a id=arc.html.get href=#arc.html.get>`arc.html.get`</a>

### HTTP `GET` handler that responds with `text/html`

Example:

```javascript
var arc = require('@architect/functions')

function handler(req, res) {
  res({
    html: '<strong>Hello world</strong>'
  })
}

exports.handler = arc.html.get(handler)
```

Things to understand:

- `arc.html.get` accepts one or more functions that follow Express-style middleware signature: `(req, res, next)=>`
- `req` is a plain JavaScript `Object` with `path`, `method`, `query`, `params`, `body` keys
- `res` is a function that must be invoked with named params: 
  - `html` a string value containing html content
  - or `location` with a URL value (a string starting w `/`)
  - `session` (optional) a plain `Object`
  - `status` (optional) HTTP error status code responses: `500`, `403`, or `404`
- `res` can also be invoked with an `Error`
  - optionally the `Error` instance property of `code`, `status` or `statusCode` can be one of `403`, `404` or `500` to change the HTTP status code
- `next` (optional) is a function to continue middleware execution 

Here's an example using `session` and `location`. First we render a form:

```javascript
// src/html/get-index
var arc = require('@architect/functions')

var form = `
<form action=/count method=post>
  <button>1up</button>
</form>
`

function handler(req, res) {
  var count = req.session.count || 0
  res({
    html: `<h1>${count}</h1><section>${form}</section>`
  })
}

exports.handler = arc.html.get(handler)

```

The form handler increments `req.session.count` and redirects back home.

```javascript
// src/html/post-count
var arc = require('@architect/functions')

function handler(req, res) {
  var count = (req.session.count || 0) + 1
  res({
    session: {count},
    location: '/'
  })
}

exports.handler = arc.html.post(handler)

```

---

## <a id=arc.html.post href=#arc.html.post>`arc.html.post`</a>

### HTTP `POST` handler that responds with HTTP status code `302` and Location redirect

- HTTP `POST` routes can **only** call `res` with `location` key and value of the path to redirect to. 
- `session` can also optionally be set.

In the following example we define `validate` middleware:

```javascript
var arc = require('@smallwins/arc-prototype')
var sendEmail = require('./_send-email')

function validate(req, res, next) {
  var isValid = typeof req.body.email != 'undefined'
  if (isValid) {
    next()
  }
  else {
    res({
      session: {
        errors: ['email missing']
      },
      location: '/contact'
    })
  }
}

function handler(req, res) {
  sendEmail({
    email: req.body.email
  }, 
  function _email(err) {
    res({
      location: `/contact?success=${err? 'yep' : 'ruhroh'}`
    })
  })
}

exports.handler = arc.html.post(validate, handler)
```

### Sessions

By default, all `@html` routes are session-enabled. If you wish to disable sessions, remove `SESSION_TABLE_NAME` env variable from the deployment config in the AWS Console.

### Errors

By default all unhandled `Error` conditions are propagated through the HTTP response. If you wish to customize the error template add a function in the Lambda root named `error.js` that accepts an `Error` and responds with a non empty `String`.

---

## <a id=arc.json.get href=#arc.json.get>`arc.json.get`</a>

Example `@json` route handler:

```javascript
var arc = require('@smallwins/arc-prototype')

function handler(req, res) {
  res({
    json: {noteID: 1, body: 'hi'}
  })
}

exports.handler = arc.json.get(handler)
```

Things to understand:

- `arc.json.get` and `arc.json.post` accept one or more functions that follow Express-style middleware signature: `(req, res, next)=>`
- `req` is a plain object with `path`, `method`, `query`, `params`, and `body` keys
- `res` is a function that must be invoked with named params: 
  - `json` a plain `Object` value
  - or `location` with a URL value (a string starting w `/`)
  - `session` (optional) a plain `Object`
  - `status` (optional) HTTP error status code responses: `500`, `403`, or `404`
- `res` can also be invoked with an `Error`
  - optionally the `Error` instance property of `code`, `status` or `statusCode` can be one of `403`, `404` or `500` to change the HTTP status code
- `next` is an optional function to continue middleware execution

### Sessions

By default, all `@json` routes are session-enabled. If you wish to disable sessions remove `SESSION_TABLE_NAME` env variable from the deployment config in the AWS Console.

### Errors

By default all unhandled `Error` conditions are propagated through the HTTP response. If you wish to customize the error template add a function in the Lambda root named `error.js` that accepts an `Error` and responds with a non empty `String`.

---

## <a id=arc.json.post href=#arc.json.post>`arc.json.post`</a>

```javascript
var arc = require('@smallwins/arc-prototype')

function handler(req, res) {
  res({
    json: {noteID: 1, body: 'hi'}
  })
}

exports.handler = arc.json.post(handler)
```

---

## <a id=arc.events.subscribe href=#arc.events.subscribe>`arc.events.subscribe`</a>

### Subscribe functions to events

An example of a `hit-counter` event handler:

```javascript
var arc = require('@smallwins/arc-prototype')

function count(payload, callback) {
  console.log(JSON.stringify(payload, null, 2))
  // maybe save count to the db here
  callback()
}

exports.handler = arc.events.subscribe(count)
```

---

## <a id=arc.events.publish href=#arc.events.publish>`arc.events.publish`</a>

### Publish events from any other function

Once deployed you can invoke `@event` handlers from any other function defined under the same `@app` namespace:

```javascript
var arc = require('@architect/functions')

arc.events.publish({
  name: 'hit-counter',
  payload: {hits: 1},
}, console.log)
```

You can also invoke Lambdas across `@app` namespaces:

```javascript
var arc = require('@architect/functions')

arc.events.publish({
  app: 'some-other-app',
  name: 'hit-counter',
  payload: {hits: 2},
}, console.log)
```

---

## <a id=arc.tables.insert href=#arc.tables.insert>`arc.tables.insert`</a>

### Respond to data being inserted into a DynamoDB table

```javascript
var arc = require('@architect/functions')

function handler(record, callback) {
  console.log(JSON.stringify(record, null, 2))
  callback()
}

exports.handler = arc.tables.insert(handler)
```

---

## <a id=arc.tables.update href=#arc.tables.update>`arc.tables.update`</a>

### Respond to data being updated in a DynamoDB table

```javascript
var arc = require('@architect/functions')

function handler(record, callback) {
  console.log(JSON.stringify(record, null, 2))
  callback()
}

exports.handler = arc.tables.update(handler)
```
---

## <a id=arc.tables.destroy href=#arc.tables.destroy>`arc.tables.destroy`</a>

### Respond to data being removed from a DynamoDB table

```javascript
var arc = require('@architect/functions')

function handler(record, callback) {
  console.log(JSON.stringify(record, null, 2))
  callback()
}

exports.handler = arc.tables.destroy(handler)
```
---

## Next: [`@app` namespace in an `.arc` file](/reference/app)
