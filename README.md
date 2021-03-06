# Redis Cache
[![Build Status](https://travis-ci.org/idealley/feathers-hooks-rediscache.png?branch=master)](https://travis-ci.org/idealley/feathers-hooks-rediscache)
[![Coverage Status](https://coveralls.io/repos/github/idealley/feathers-hooks-rediscache/badge.svg?branch=master)](https://coveralls.io/github/idealley/feathers-hooks-rediscache?branch=master)

> Cache any route with redis

## Installation

```
npm install feathers-hooks-rediscache --save
```

## Purpose
The purpose of these hooks is to provide redis caching for APIs enpoints. 

Each request to an endpoint can be cached. Route variables and params are cached on a per request base. If a param to call is set to true and then to false to responses will be cached.

The cache can be purged for an individual route, but also for a group of routes. This is very useful if you have an API endpoint that creates a list of articles, and a endpoint that returns an individual article. If the article is modified, the list of articles should, most likely, be purged as well. This can be done calling one endpoint.

In the same fashion if you have many variant of the same endpoint that return similar content based on parameters you can bust the whole group as well:

```js
/articles // list
/articles/article //individual item
/articles/article?markdown=true // variant
```

These are all listed in a redis list under `group-articles` and can be busted by calling `/cache/clear/group/article` or `/cache/clear/group/articles` it does not matter. All urls will be purged. 

It was meant to be used over http, not tested with sockets.

## Documentation
Add the different hooks. The order matters (see bellow). A `cache` object will be added to your response. This is usefull has other systems can use this object to purge the cache if needed.

availble routes:
```js
/cache/index // returns an array with all the keys
/cache/clear // clears the whole cache
/cache/clear/single/:target // clears a single route if you want to purge a route with params just adds them target?param=1
/cache/clear/group/:target // clears a group
```

## Complete Example

Here's an example of a Feathers server that uses `feathers-hooks-rediscache`. 

```js
const feathers = require('feathers');
const rest = require('feathers-rest');
const hooks = require('feathers-hooks');
const bodyParser = require('body-parser');
const errorHandler = require('feathers-errors/handler');
const routes = require('feathers-hooks-rediscache');

// Initialize the application
const app = feathers()
  .configure(rest())
  .configure(hooks())
  // Needed for parsing bodies (login)
  .use(bodyParser.json())
  .use(bodyParser.urlencoded({ extended: true }))
  // add the cache routes (endpoints) to the app
  .use('/cache', routes)
  .use(errorHandler());

app.listen(3030);

console.log('Feathers app started on 127.0.0.1:3030');
```

Add hooks on the routes that need caching
```js
//services/<service>.hooks.js

const redisBefore = require('feathers-hooks-rediscache').redisBeforeHook;
const redisAfter = require('feathers-hooks-rediscache').redisAfterHook;
const cache = require('feathers-hooks-rediscache').hookCache;


module.exports = {
  before: {
    all: [],
    find: [redisBefore()],
    get: [redisBefore()],
    create: [],
    update: [],
    patch: [],
    remove: []
  },

  after: {
    all: [],
    find: [cache({duration: 3600 * 24 * 7}), redisAfter()],
    get: [cache({duration: 3600 * 24 * 7}), redisAfter()],
    create: [],
    update: [],
    patch: [],
    remove: []
  },

  error: {
    all: [],
    find: [],
    get: [],
    create: [],
    update: [],
    patch: [],
    remove: []
  }
};
```
* the duration is in seconds and will automatically expire
* you may just use `cache()` with out specifying a duration, any request will be cached for a day

## to does
* add configuration for the default duration
* add configuration for the redis db (now using defaults) 

## License

Copyright (c) 2017

Licensed under the [MIT license](LICENSE).
