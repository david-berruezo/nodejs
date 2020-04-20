# url

## Overview

The url module provides utilities for URL resolution and parsing. It can be accessed using:

```js
const url = require('url');
```

```js
const myURL = new URL('/foo', 'https://example.org/');
// https://example.org/foo
```

```js
const myURL = new URL({ toString: () => 'https://example.org/' });
// https://example.org/
```

## Options

### url.hash

Gets and sets the fragment portion of the URL.

```js
const myURL = new URL('https://example.org/foo#bar');
console.log(myURL.hash);
// Prints #bar

myURL.hash = 'baz';
console.log(myURL.href);
// Prints https://example.org/foo#baz
```

### url.host

Gets and sets the host portion of the URL.

```js
const myURL = new URL('https://example.org:81/foo');
console.log(myURL.host);
// Prints example.org:81

myURL.host = 'example.com:82';
console.log(myURL.href);
// Prints https://example.com:82/foo
```

### url.hostname

Gets and sets the host name portion of the URL. The key difference between url.host and url.hostname is that url.hostname does not include the port.

```js
const myURL = new URL('https://example.org:81/foo');
console.log(myURL.hostname);
// Prints example.org

myURL.hostname = 'example.com:82';
console.log(myURL.href);
// Prints https://example.com:81/foo
```

### url.href

```js
const myURL = new URL('https://example.org/foo');
console.log(myURL.href);
// Prints https://example.org/foo

myURL.href = 'https://example.com/bar';
console.log(myURL.href);
// Prints https://example.com/bar
```

### url.origin

Gets the read-only serialization of the URL's origin.

```js
const myURL = new URL('https://example.org/foo/bar?baz');
console.log(myURL.origin);
// Prints https://example.org
```

```js
const idnURL = new URL('https://測試');
console.log(idnURL.origin);
// Prints https://xn--g6w251d

console.log(idnURL.hostname);
// Prints xn--g6w251d
```

.... and more ...