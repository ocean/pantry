# Pantry

[![Build Status](https://travis-ci.org/ocean/pantry.svg)](https://travis-ci.org/ocean/pantry)

A JSON/XML resource caching library based on Request

## Introduction

Pantry is an HTTP client cache used to minimize requests for external JSON and XML feeds.  Pantry will cache resources to minimize round trips, utilizing the local cache when available and refresh the cache (asynchronously) as needed.  (As of 0.7.x, Pantry can also proxy cache any raw resource.) 

As with any of our projects, constructive criticism is encouraged.

## Installing

Just grab [node.js](http://nodejs.org/#download) and [npm](http://github.com/isaacs/npm) and you're set:

	npm install pantry
	
Pantry uses the amazing [Request](https://github.com/mikeal/request) library for all HTTP requests and [xml2js](https://github.com/Leonidas-from-XIV/node-xml2js) to parse xml resources.

## Using

Work in progress.  See the examples for the time being.

To utilize the pantry, simply require it, optionally override some default values, and then request your resource(s) via the fetch method

	var pantry = require('pantry');
	pantry.configure({ shelfLife: 5 });
	
	pantry.fetch({ uri: 'http://search.twitter.com/search.json?q=winning'}, function (error, item, contentType) {
		console.log(item.results[0].text);
	});

At this item, the following configuration options can be specified:

* shelfLife - number of seconds before a resource reaches it's best-before-date
* maxLife - number of seconds before a resource spoils
* caseSensitive - URI should be considered case sensitive when inferring cache key
* verbosity - possible values are 'silly', 'debug, 'verbose', info', 'warn' and 'error'  (default is 'error' for production systems)
* parser - possible values are 'json', 'xml', or 'raw'  (default is undefined, in which auto-detection by content-type header is attempted)
* ignoreCacheControl - do not utilize the cache-control header to override the cache configuration if present (default is false)
* cacheBuster - adds the provided query string parameter name with a cachebusting timestamp to the request
* xmlOptions - options passed into the [xml2js](https://github.com/Leonidas-from-XIV/node-xml2js) parser (default is  {explicitRoot: false})

When you request a resource from the pantry, a couple interesting things happen.  If the item is available in the pantry, and hasn't 'spoiled', it will be returned immediately via the callback.  If it has expired (it's beyond its best before date) but hasn't spoiled, it will still be returned and then refreshed in the background.

> Ode to my immigrant mother:  The best before date is treated as a recommendation.  If it hasn't visibly spoiled it's probably still good to use, so use it until we have a chance to go shopping.  Especially if it's salad dressing, that stuff never goes bad as long as it's in the fridge.

If the resource isn't available in the pantry, or the item has spoiled, then the item will be retrieved immediately and won't be passed on to the callback method until we have the resource on hand.

Pantry will also ensure that we don't fetch the same resource multiple times in parallel.  If a resource is already being requested, additional requests for that same resource will hook into the same completion event for the original request.

If you wish to manually remove an item from the pantry, you can do this by calling the `remove()` method with the URI of the resource you wish to remove:

    pantry.remove('http://search.twitter.com/search.json?q=winning');

## Storage

The latest version of Pantry ( >= 0.3 ) supports the ability to plug the caching storage engine of your choice.  By default, pantry will utilize the MemoryStorage plugin, which will (of all things) cache items locally in memory.  

To specify an alternate storage engine, or to provide custom configuration for the default memory storage, simply assign the new storage engine to pantry via the .storage property.

### MemoryStorage

The constructor for MemoryStorage takes two parameters:

* config - hash of configuration properties (see below)
* verbosity - controls the level of logging (default is 'info')

The following configuration properties are allowed for MemoryStorage

* capacity - the maximum number of items to keep in the pantry (default is 1000)
* ideal - when cleaning up, the ideal number of items to keep in the pantry (default is 90% of capacity)


Example:

	var pantry = require('pantry');
		, MemoryStorage = require('pantry/lib/pantry-memory');

	pantry.storage = new MemoryStorage({
	  capacity: 18,
	  ideal: 12
	}, 'debug');

Note that the ideal must be set to a value which is between 10% and 90% of capacity.  Every time an item is added to pantry, we ensure we haven't reached capacity.  If we have, then we first start with throwing out any spoiled items.  After that, if we are still above capacity we will get rid of the expired items, and if we're really desperate we will need to throw out some good items just to make room.

### RedisStorage

A simply plugin for Redis is included with Pantry.  Note that since use of Redis is optional, the required client (redis) is not included in the package dependencies.  You must include it in your own application's dependencies and/or manually install it via npm install redis

The constructor for RedisStorage takes four parameters:

* port - the redis server port (default is 6379)
* host - the redis server host name (default is 'localhost')
* options - hash of configuration properties (see below)
* verbosity - controls the level of logging (default is 'info')

The following configuration properties are allowed for RedisStorage

* auth - the password / authentication key for the redis server (default is null)

Example:

	var pantry = require('pantry')
		, RedisStorage = require('pantry/lib/pantry-redis');

	pantry.storage = new RedisStorage(6379, 'localhost', null, 'debug');

### MemcachedStorage

A simply plugin for Memcached is also included with Pantry.  Note that since use of Memcached is optional, the required client (memcached) is not included in the package dependencies.  You must include it in your own application's dependencies and/or manually install it via npm install memcached

The constructor for MemcachedStorage takes three parameters:

* servers - a string or array of strings identifying the Memcached server(s) to use
* options - hash of memcached configuration properties (see [here](https://github.com/3rd-Eden/node-memcached#readme) for more details)
* verbosity - controls the level of logging (default is 'info')

Example:

	var pantry = require('../src/pantry')
		, MemcachedStorage = require('../src/pantry-memcached');

	pantry.storage = new MemcachedStorage('localhost:11211', {}, 'debug');
	

## SOAP Support

Support for SOAP requests has been dropped from this version due to the many security issues with dependencies, and the fact that I don't need SOAP support.
	
## Upgrading

As of v0.4.x, we now use v0.2.x of the xml2js library.  This has significantly changed the default parsing options.  You can easily revert to the xml2js v0.1 parsing options as described [here](https://github.com/Leonidas-from-XIV/node-xml2js)

## Roadmap

* Better handling of not-GET requests
* Ability to execute array of requests in parallel
* Support for cookies (including cache key)

## Created and managed by

* Edward de Groot
* Keith Benedict
