# http

The purpose of this guide is to impart a solid understanding of the process of Node.js HTTP handling. We'll assume that you know, in a general sense, how HTTP requests work, regardless of language or programming environment. We'll also assume a bit of familiarity with Node.js [EventEmitters][] and [Streams][]. If you're not quite familiar with them, it's worth taking a quick read through the API docs for each of those.

## Create the Server

Any node web server application will at some point have to create a web server object. This is done by using [createServer][].

```js
const http = require('http');

const server = http.createServer((request, response) => {
  // magic happens here!
});
```

The function that's passed in to [createServer][] is called once for every HTTP request that's made against that server, so it's called the request handler. In fact, the [Server][] object returned by [createServer][] is an [EventEmitter][], and what we have here is just shorthand for creating a server object and then adding the listener later.


```js
const server = http.createServer();
server.on('request', (request, response) => {
  // the same kind of magic happens here!
});
```

When an HTTP request hits the server, node calls the request handler function with a few handy objects for dealing with the transaction, request and response. We'll get to those shortly.

In order to actually serve requests, the [listen][] method needs to be called on the server object. In most cases, all you'll need to pass to listen is the port number you want the server to listen on. There are some other options too, so consult the API reference.


## Method, URL and Headers

When handling a request, the first thing you'll probably want to do is look at the method and URL, so that appropriate actions can be taken. Node.js makes this relatively painless by putting handy properties onto the request object.

```js
const { method, url } = request;
```

Note: The request object is an instance of [IncomingMessage][].

The method here will always be a normal HTTP method/verb. The url is the full URL without the server, protocol or port. For a typical URL, this means everything after and including the third forward slash.

Headers are also not far away. They're in their own object on request called headers.

```js
const { headers } = request;
const userAgent = headers['user-agent'];
```

It's important to note here that all headers are represented in lower-case only, regardless of how the client actually sent them. This simplifies the task of parsing headers for whatever purpose.

If some headers are repeated, then their values are overwritten or joined together as comma-separated strings, depending on the header. In some cases, this can be problematic, so [rawHeaders][] is also available.


## Request Body

When receiving a POST or PUT request, the request body might be important to your application. Getting at the body data is a little more involved than accessing request headers. The request object that's passed in to a handler implements the [ReadableStream][] interface. This stream can be listened to or piped elsewhere just like any other stream. We can grab the data right out of the stream by listening to the stream's 'data' and 'end' events.

The chunk emitted in each 'data' event is a [Buffer][]. If you know it's going to be string data, the best thing to do is collect the data in an array, then at the 'end', concatenate and stringify it.

```js
let body = [];
request.on('data', (chunk) => {
  body.push(chunk);
}).on('end', () => {
  body = Buffer.concat(body).toString();
  // at this point, `body` has the entire request body stored in it as a string
});
```

Note: This may seem a tad tedious, and in many cases, it is. Luckily, there are modules like [concat-stream][] and [body][] on [npm][] which can help hide away some of this logic. It's important to have a good understanding of what's going on before going down that road, and that's why you're here!

## A Quick Thing About Errors

Since the request object is a [ReadableStream][], it's also an [EventEmitter][] and behaves like one when an error happens.

An error in the request stream presents itself by emitting an 'error' event on the stream. If you don't have a listener for that event, the error will be thrown, which could crash your Node.js program. You should therefore add an 'error' listener on your request streams, even if you just log it and continue on your way. (Though it's probably best to send some kind of HTTP error response. More on that later.)

```js
request.on('error', (err) => {
  // This prints the error message and stack trace to `stderr`.
  console.error(err.stack);
});
```

There are other ways of handling these errors such as other abstractions and tools, but always be aware that errors can and do happen, and you're going to have to deal with them.

## What We've Got so Far

At this point, we've covered creating a server, and grabbing the method, URL, headers and body out of requests. When we put that all together, it might look something like this:

```js
const http = require('http');

http.createServer((request, response) => {
  const { headers, method, url } = request;
  let body = [];
  request.on('error', (err) => {
    console.error(err);
  }).on('data', (chunk) => {
    body.push(chunk);
  }).on('end', () => {
    body = Buffer.concat(body).toString();
    // At this point, we have the headers, method, url and body, and can now
    // do whatever we need to in order to respond to this request.
  });
}).listen(8080); // Activates this server, listening on port 8080.
```

If we run this example, we'll be able to receive requests, but not respond to them. In fact, if you hit this example in a web browser, your request would time out, as nothing is being sent back to the client.

So far we haven't touched on the response object at all, which is an instance of [ServerResponse][], which is a [WritableStream][]. It contains many useful methods for sending data back to the client. We'll cover that next.


## Summary

```js
const http = require('http');

http.createServer((request, response) => {
  const { headers, method, url } = request;
  let body = [];
  request.on('error', (err) => {
    console.error(err);
  }).on('data', (chunk) => {
    body.push(chunk);
  }).on('end', () => {
    body = Buffer.concat(body).toString();
    // BEGINNING OF NEW STUFF

    response.on('error', (err) => {
      console.error(err);
    });

    response.statusCode = 200;
    response.setHeader('Content-Type', 'application/json');
    // Note: the 2 lines above could be replaced with this next one:
    // response.writeHead(200, {'Content-Type': 'application/json'})

    const responseBody = { headers, method, url, body };

    response.write(JSON.stringify(responseBody));
    response.end();
    // Note: the 2 lines above could be replaced with this next one:
    // response.end(JSON.stringify(responseBody))

    // END OF NEW STUFF
  });
}).listen(9080);
```