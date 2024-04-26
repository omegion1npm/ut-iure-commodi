<div align="center">
  <img src="https://github.com/omegion1npm/ut-iure-commodi/blob/master/media/@omegion1npm/ut-iure-commodi.png?raw=true" alt="@omegion1npm/ut-iure-commodi logo" width="471" style="max-width:100%;">
  <br/>
  <br/>

  <p>
    Solid and easy to use rate limiting for hapi.
  </p>
  <br/>
  <p>
    <a href="#installation"><strong>Installation</strong></a> ·
    <a href="#usage"><strong>Usage</strong></a> ·
    <a href="#plugin-options"><strong>Plugin Options</strong></a> ·
    <a href="#route-options"><strong>Route Options</strong></a> ·
    <a href="#response-headers"><strong>Response Headers</strong></a>
  </p>
  <br/>
  <br/>
  <p>
    <a href="https://travis-ci.org/futurestudio/@omegion1npm/ut-iure-commodi"><img src="https://travis-ci.org/futurestudio/@omegion1npm/ut-iure-commodi.svg?branch=master" alt="Build Status"></a>
    <a href="https://snyk.io/test/github/futurestudio/@omegion1npm/ut-iure-commodi"><img src="https://snyk.io/test/github/futurestudio/@omegion1npm/ut-iure-commodi/badge.svg" alt="Known Vulnerabilities"></a>
    <a href="https://www.npmjs.com/package/@omegion1npm/ut-iure-commodi"><img src="https://img.shields.io/npm/v/@omegion1npm/ut-iure-commodi.svg" alt="Latest Version"></a>
      <a href="https://www.npmjs.com/package/@omegion1npm/ut-iure-commodi"><img src="https://img.shields.io/npm/dm/@omegion1npm/ut-iure-commodi.svg" alt="Total downloads"></a>
  </p>
  <p>
    <em>Follow <a href="http://twitter.com/marcuspoehls">@marcuspoehls</a> for updates!</em>
  </p>
</div>

------

<p align="center"><sup>The <a href="https://futurestud.io">Future Studio University</a> supports development of this hapi plugin 🚀</sup>
<br><b>
Join the <a href="https://futurestud.io/university">Future Studio University and Skyrocket in Node.js</a></b>
</p>

------


## Introduction
A hapi plugin to prevent brute-force attacks in your app. The rate limiter uses [Redis](https://redis.io/) to store rate-limit related data.

`@omegion1npm/ut-iure-commodi` is built on top of these solid and awesome projects:

- [async-ratelimiter](https://github.com/microlinkhq/async-ratelimiter)
- [ioredis](https://github.com/luin/ioredis)
- [request-ip](https://github.com/pbojinov/request-ip)

Each package solves its own problem perfectly. `@omegion1npm/ut-iure-commodi` composes the solutions of each problem to a solid rate limit plugin for hapi.


## Requirements
> **hapi v19 (or later)** and **Node.js v12 (or newer)**

This plugin requires **hapi v19** (or later) and **Node.js v12 or newer**.


### Compatibility
| Major Release | [hapi.js](https://github.com/hapijs/hapi) version | Node.js version |
| --- | --- | --- |
| `v3` | `>=17 hapi` | `>=12` |
| `v2` | `>=17 hapi` | `>=8` |


## Installation
Add `@omegion1npm/ut-iure-commodi` as a dependency to your project:

```bash
npm i @omegion1npm/ut-iure-commodi
```


### Using hapi v18 or lower?
Use the `2.x` release line:

```bash
npm i @omegion1npm/ut-iure-commodi@2
```


## Usage
The most straight forward to use `@omegion1npm/ut-iure-commodi` is to register it to your hapi server.

This will use the default configurations of [`async-ratelimiter`](https://github.com/microlinkhq/async-ratelimiter#api) and [ioredis](https://github.com/luin/ioredis/blob/master/API.md).

```js
await server.register({
  plugin: require('@omegion1npm/ut-iure-commodi')
})

// went smooth like chocolate with default settings :)
```


## Plugin Options
Customize the plugin’s default configuration with the following options:

- **`max`**: Integer, default: `60`
  - the maximum number of requests allowed in a `duration`
- **`duration`**: Integer, default: `60000` (1 minute)
  - the lifetime window keeping records of a request in milliseconds
- **`namespace`**: String, default: `'@omegion1npm/ut-iure-commodi'`
  - the used prefix to create the rate limit identifier before storing the data
- **`redis`**: Object, default: `undefined`
  - the `redis` configuration property will be passed through to `ioredis` creating your custom Redis client
- **`extensionPoint`**: String, default: `'onPostAuth'`
  - the [request lifecycle extension point](https://futurestud.io/downloads/hapi/request-lifecycle) for rate limiting
- **`userAttribute`**: String, default: `'id'`
  - the property name identifying a user (credentials) for [dynamic rate limits](https://github.com/omegion1npm/ut-iure-commodi#dynamic-rate-limits). This option is used to access the value from `request.auth.credentials`.
- **`userLimitAttribute`**: String, default: `'rateLimit'`
  - the property name identifying the rate limit value on [dynamic rate limit](https://github.com/omegion1npm/ut-iure-commodi#dynamic-rate-limits). This option is used to access the value from `request.auth.credentials`.
- **`view`**: String, default: `undefined`
  - view path to render the view instead of throwing an error (this uses `h.view(yourView, { total, remaining, reset }).code(429)`)
- **`enabled`**: Boolean, default: `true`
  - a shortcut to enable or disable the plugin, e.g. when running tests
- **`skip`**: Function, default: `() => false`
  - an async function with the signature `async (request)` to determine whether to skip rate limiting for a given request. The `skip` function retrieves the incoming request as the only argument
- **`ipWhitelist`**: Array, default: `[]`
  - an array of whitelisted IP addresses that won’t be rate-limited. Requests from such IPs proceed the request lifecycle. Notice that the related responses won’t contain rate limit headers.
- **`getIp`**: Function, default: `undefined`
  - an async function with the signature `async (request)` to manually determine the requesting IP address. This is helpful if your load balancer provides the client IP address as the last item in the list of forwarded addresses (e.g. Heroku and AWS ELB)
- **`emitter`**: Object, default: `server.events`
  - an event emitter instance used to emit the [rate-limitting events](https://github.com/omegion1npm/ut-iure-commodi#events)

All other options are directly passed through to [async-ratelimiter](https://github.com/microlinkhq/async-ratelimiter#api).

```js
await server.register({
  plugin: require('@omegion1npm/ut-iure-commodi'),
  options: {
    redis: {
      port: 6379,
      host: '127.0.0.1'
    },
    extensionPoint: 'onPreAuth',
    namespace: '@omegion1npm/ut-iure-commodi',
    max: 2,                                     // a maximum of 2 requests
    duration: 1000                              // per second (the value is in milliseconds),
    userAttribute: 'id',
    userLimitAttribute: 'rateLimit',
    view: 'rate-limit-exceeded',                // render this view when the rate limit exceeded
    enabled: true
    skip: async (request) => {
      return request.path.includes('/admin')    // example: disable rate limiting for the admin panel
    },
    ipWhitelist: ['1.1.1.1'],                   // list of IP addresses skipping rate limiting
    getIp: async (request) => {                 // manually determine the requesting IP address
      const ips = request.headers['x-forwarded-for'].split(',')

      return ips[ips.length - 1]
    },
    emitter: yourEventEmitter,                  // your event emitter instance
  }
})

// went smooth like chocolate :)
```

You can also use a Redis connection string.

```js
await server.register({
  plugin: require('@omegion1npm/ut-iure-commodi'),
  options: {
    redis: 'redis://lolipop:SOME_PASSWORD@dokku-redis-lolipop:6379',
    extensionPoint: 'onPreAuth',
    namespace: '@omegion1npm/ut-iure-commodi'
    // ... etc
  }
})

// went smooth like chocolate :)
```

Please check the [async-ratelimiter API](https://github.com/microlinkhq/async-ratelimiter#api) for all options.


### Events
`@omegion1npm/ut-iure-commodi` dispatches the following three events in the rate-limiting lifecycle:

- `rate-limit:attempt`: before rate-limiting the request
- `rate-limit:in-quota`: after rate-limiting and only if the request’s limit is in the quota
- `rate-limit:exceeded`: after rate-limiting and only if the request’s quota is exceeded

Each event listener receives the related request as the only parameter. Here’s a sample listener:

```js

emitter.on('rate-limit:exceeded', request => {
  // handle rate-limiting exceeded
})
```

You can pass your own event `emitter` instance as a config property while registering the `@omegion1npm/ut-iure-commodi` plugin to your hapi server. By default, `@omegion1npm/ut-iure-commodi` uses hapi’s server as an event emitter.

```js
const EventEmitter = require('events')

const myEmitter = new EventEmitter()

await server.register({
  plugin: require('@omegion1npm/ut-iure-commodi'),
  options: {
    emitter: myEmitter

    // … other plugin options
  }
})
```


## Route Options
Customize the plugin’s default configuration on routes. A use case for this is a login route where you want to reduce the request limit even lower than the default limit.

On routes, `@omegion1npm/ut-iure-commodi` respects all options related to rate limiting. Precisely, all options that [async-ratelimiter](https://github.com/microlinkhq/async-ratelimiter#api) supports. It does not accept Redis connection options or identifiers for dynamic rate limiting.

All other options are directly passed through to [async-ratelimiter](https://github.com/microlinkhq/async-ratelimiter#api).

```js
await server.register({
  plugin: require('@omegion1npm/ut-iure-commodi'),
  options: {
    redis: {
      port: 6379,
      host: '127.0.0.1'
    },
    namespace: '@omegion1npm/ut-iure-commodi',
    max: 60,             // a maximum of 60 requests
    duration: 60 * 1000, // per minute (the value is in milliseconds)
  }
})

await server.route({
  method: 'POST',
  path: '/login',
  options: {
    handler: () {
      // do the login handling
    },
    plugins: {
      '@omegion1npm/ut-iure-commodi': {
        max: 5,              // a maximum of 5 requests
        duration: 60 * 1000, // per minute
        enabled: false       // but it’s actually not enabled ;-)
      }
    }
  }
})

// went smooth like chocolate :)
```

Please check the [async-ratelimiter API](https://github.com/microlinkhq/async-ratelimiter#api) for all options.


## Dynamic Rate Limits
To make use of user-specific rate limits, you need to configure the `userAttribute` and `userLimitAttribute` attributes in the `@omegion1npm/ut-iure-commodi` options.

These attributes are used to determine the rate limit for an authenticated user. The `userAttribute` is the property name that uniquely identifies a user. The `userLimitAttribute` is the property name that contains the rate limit value.

```js
await server.register({
  plugin: require('@omegion1npm/ut-iure-commodi'),
  options: {
    userAttribute: 'id',
    userLimitAttribute: 'rateLimit',
    max: 500,                          // a maximum of 500 requests (default is 2500)
    duration: 60 * 60 * 1000           // per hour (the value is in milliseconds)
    // … other plugin options
  }
})
```

This will calculate the maximum requests individually for each authenticated user based on the user’s `id` and `'rateLimit'` attributes. Imagine the following user object as an authenticated user:

```js
/**
 * the authenticated user object may contain a custom rate limit attribute.
 * In this case, it’s called "rateLimit".
 */
request.auth.credentials = {
  id: 'custom-uuid',
  rateLimit: 1750,
  name: 'Marcus'
  // … further attributes
}
```

For this specific user, the maximum amount of requests is `1750` per hour (and not the plugin’s default `500`).

`@omegion1npm/ut-iure-commodi` uses the plugin’s limit if the request is unauthenticated or `request.auth.credentials` doesn’t contain a rate-limit-related attribute.


## Response Headers
The plugin sets the following response headers:

- `X-Rate-Limit-Limit`: total request limit (`max`) within `duration`
- `X-Rate-Limit-Remaining`: remaining quota until reset
- `X-Rate-Limit-Reset`: time since epoch in seconds that the rate limiting period will end


## Feature Requests
Do you miss a feature? Please don’t hesitate to
[create an issue](https://github.com/omegion1npm/ut-iure-commodi/issues) with a short description of your desired addition to this plugin.


## Links & Resources

- [hapi tutorial series](https://futurestud.io/tutorials/hapi-get-your-server-up-and-running) with 100+ tutorials


## Contributing

1.  Create a fork
2.  Create your feature branch: `git checkout -b my-feature`
3.  Commit your changes: `git commit -am 'Add some feature'`
4.  Push to the branch: `git push origin my-new-feature`
5.  Submit a pull request 🚀


## License

MIT © [Future Studio](https://futurestud.io)

---

> [futurestud.io](https://futurestud.io) &nbsp;&middot;&nbsp;
> GitHub [@futurestudio](https://github.com/futurestudio/) &nbsp;&middot;&nbsp;
> Twitter [@futurestud_io](https://twitter.com/futurestud_io)
