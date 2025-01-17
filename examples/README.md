# JavaScript Client SDK

_Read this in other languages_: [日本語](README.ja.md)

<br>

## Running the Example

- Node version 16 or higher is required
- A Momento Auth Token is required, you can generate one using the [Momento CLI](https://github.com/momentohq/momento-cli)

```bash
npm install

# Run example code
MOMENTO_AUTH_TOKEN=<YOUR AUTH TOKEN> npm run example
```

Example Code: [index.ts](index.ts)

## Running the Presigned URL Example

This example demonstrates how to use the `MomentoSigner` class, which enables creation of user-signed access tokens and presigned urls which can be used for doing cache get/set actions via HTTP.

- Node version 16 or higher is required
- A Momento Auth Token is required, you can generate one using the [Momento CLI](https://github.com/momentohq/momento-cli)

```bash
npm install

# Run example code
MOMENTO_AUTH_TOKEN=<YOUR AUTH TOKEN> npm run presigned-url-example
```

Example Code: [presigned-url-example.ts](presigned-url-example.ts)

## Using the SDK in your projects

### Installation

```bash
npm install @gomomento/sdk
```

### Usage

```typescript
import {
  SimpleCacheClient,
  CacheGet,
  CacheSet,
  CreateCache,
} from '@gomomento/sdk';

const main = async () => {
  // your authentication token for momento
  const authToken = process.env.MOMENTO_AUTH_TOKEN;
  if (!authToken) {
    throw new Error('Missing required environment variable MOMENTO_AUTH_TOKEN');
  }

  // initializing momento
  const DEFAULT_TTL = 60; // 60 seconds for default ttl
  const momento = new SimpleCacheClient(authToken, DEFAULT_TTL);

  // creating a cache named "myCache"
  const CACHE_NAME = 'myCache';
  const createResponse = await momento.createCache(CACHE_NAME);
  if (createResponse instanceof CreateCache.Error) {
    console.log('Error creating cache');
    throw createResponse.innerException();
  }
  // sets key with default ttl
  const setResponse = await momento.set(CACHE_NAME, 'key', 'value');
  if (setResponse instanceof CacheSet.Error) {
    console.log('Error setting cache key');
  }

  const getResponse = await momento.get(CACHE_NAME, 'key');
  if (getResponse instanceof CacheGet.Hit) {
    console.log('result: ', getResponse.valueString());
  } else if (getResponse instanceof CacheGet.Miss) {
    console.log('Got a cache miss');
  } else if (getResponse instanceof CacheGet.Error) {
    console.log(`Got an error: ${getResponse.message()}`);
  }
  // sets key with ttl of 5 seconds
  const ttlSetResponse = await momento.set(CACHE_NAME, 'key2', 'value2', 5);

  // permanently deletes cache
  const deleteResponse = await momento.deleteCache(CACHE_NAME);
};

main()
  .then(() => {
    console.log('success!!');
  })
  .catch(e => {
    console.error('failed to get from cache', e);
  });
```

Momento also supports storing pure bytes,

```typescript
const key = new Uint8Array([109, 111, 109, 101, 110, 116, 111]);
const value = new Uint8Array([
  109, 111, 109, 101, 110, 116, 111, 32, 105, 115, 32, 97, 119, 101, 115, 111,
  109, 101, 33, 33, 33,
]);
const setResponse = await momento.set('cache', key, value, 50);
const getResponse = await momento.get('cache', key);
```

Handling cache misses

```typescript
const getResponse = await cache.get('cache', 'non-existent key');
if (getResponse instanceof CacheGet.Miss) {
  console.log('cache miss');
}
```

Storing Files

```typescript
const buffer = fs.readFileSync('./file.txt');
const filebytes = Uint8Array.from(buffer);
const cacheKey = 'key';
const cacheName = 'my example cache';

// store file in cache
const setResponse = await momento.set(cacheName, cacheKey, filebytes);

// retrieve file from cache
const getResponse = await momento.get(cacheName, cacheKey);

// write file to disk
if (getResponse instanceof CacheGet.Hit) {
  fs.writeFileSync('./file-from-cache.txt', Buffer.from(getResponse.valueBytes()));
}
```

## Running the load generator example

This repo includes a very basic load generator, to allow you to experiment with
performance in your environment based on different configurations. It's very
simplistic, and only intended to give you a quick way to explore the performance
of the Momento client running on a single nodejs process.

Note that because nodejs javascript code runs on a single thread, the limiting
factor in request throughput will often be CPU. Keep an eye on your CPU
consumption while running the load generator, and if you reach 100%
of a CPU core then you most likely won't be able to improve throughput further
without running additional nodejs processes.

CPU will also impact your client-side latency; as you increase the number of
concurrent requests, if they are competing for CPU time then the observed
latency will increase.

Also, since performance will be impacted by network latency, you'll get the best
results if you run on a cloud VM in the same region as your Momento cache.

Check out the configuration settings at the bottom of the 'load-gen.ts' to
see how different configurations impact performance.

If you have questions or need help experimenting further, please reach out to us!

To run the load generator:

```bash
# Run example load generator
MOMENTO_AUTH_TOKEN=<YOUR AUTH TOKEN> npm run load-gen
```

You can check out the example code in [load-gen.ts](load-gen.ts). The configurable
settings are at the bottom of the file.
