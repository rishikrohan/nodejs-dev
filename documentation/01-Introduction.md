# What Node?

Node.js is an asynchronous, event-driven JavaScript runtime.

Whats is a run time? A Run time is nothing but an environment / A set of standard libraries required for executing a task or a program.

For example the google chrome [V8 engine](https://v8.dev/). Written in C++, V8 is google's open source JavaScript and web-assembly engine.

Before node, JavaScript usage was confined to browser for writing dynamic frontend applications.

NodeJs also leverages the same Google Chrome's V8 engine to compile and execute JavaScript Programmes.

Java Needs JRE to compile and run Java Programmes, Similarly We need an execution context / runtime to execute JS programs and that is provided by some engine (Firefox / IE /Chrome).

## Why NodeJs?

- Great for prototyping and agile development at the same time super fast and highly scalable.
  - Nodejs at [netflix](https://youtu.be/p74282nDMX8)
  - Nodejs at [paypal](https://youtu.be/k2wkNMr9idE)
- Eventloop, Single Threaded, Asynchronous Program?? What the heck?
  - One of the strengths of node and JavaScript in general is their single threaded programming model.
  - Threads are a common source of bugs, and although some recent programming languages, including Go and Rust, have attempted to offer safer concurrency tools, Node retains the model used in the browser.
  - In browser-based code, we write sequences of instructions that execute one at a time; code doesn’t execute in parallel.
  - This doesn’t make sense for user interfaces, however: users don’t want to wait around for slow operations such as network or file access to finish. To get around this, browsers use events: when you click a button, an event fires, and a function runs that has previously been defined but not yet executed. This avoids some of the issues found in threaded programming, including resource deadlocks and race conditions.

## Core Node Modules

Node’s core modules are similar to other languages’ standard libraries; these are the tools you need to write server-side JavaScript.

- **FileSystem**: Node ships with a filesystem library (fs, path), TCP clients and servers (net), HTTP (http and https), and domain name resolution (dns). There’s a useful assertion library that’s used mostly to write tests (assert), and an operating system library for querying information about the platform (os).

  Node also has libraries that are unique to Node. The events module is a small library for working with events, and it’s used as a basis for much of Node’s APIs. For example, the stream module uses the events module to provide abstract interfaces for working with streams of data. Because all data streams in Node use the same APIs, you can easily compose software components; if you have a file-stream reader, you can pipe it through a zlib transform that compresses the data, and then pipe it through a file-stream writer to write the data out to a file.

  The following listing shows how to use Node’s fs module to create read- and write-streams that can be piped through another stream (gzip) to transform the data—in this case, by compressing it.

  ```javascript
  const fs = require('fs');
  const zlib = require('zlib');
  const gzip = zlib.createGzip();
  const outStream = fs.createWriteStream('output.js.gz');

  fs.createReadStream('./node-stream.js')
    .pipe(gzip)
    .pipe(outStream);
  ```

- **Networking**: To build a server in Node, you just need to load the http module and give it a function. The function accepts two arguments: the incoming request and the outgoing response.

  ```javascript
  const http = require('http');
  const port = 8080;

  const server = http.createServer((req, res) => {
    res.end('Hello, world.');
  });

  server.listen(port, () => {
    console.log('Server listening on: http://localhost:%s', port);
  });
  ```

## Non-Blocking I/O

- What does this mean in the context of server-side programming?
  - I/O requests such as disk and network access are also comparatively slow, so we don’t want the runtime to block business logic from executing while reading files or sending messages over the network.
  - To solve this, Node uses three techniques:
    1. Events
    2. Asynchronous APIs
    3. NonBlocking I/O : It means your program can make a request for a network resource while doing something else, and then, when the network operation has finished, a callback will run that handles the result.
    4. ![nonblocking](/Users/rishik_rohan/Documents/work/dev/code-academy/nodejs-dev/documentation/assets/nonblocking.jpg)


In the above figure, the database is accessed over the network. In Node, that network access is nonblocking, because Node uses a library called libuv (http://libuv.org/) to provide access to the operating system’s nonblocking network calls.

*Libuv: libuv is the native library that provides fast, cross-platform, nonblocking I/O.*

This is implemented differently in Linux, macOS, and Windows, but all you have to worry about is your friendly Java-Script database library. While you’re writing code such as db.insert(query, err => {}), Node is doing highly optimized, nonblocking networking underneath.

Disk access is similar, but intriguingly not the same. When the email receipt is generated and the email template is read from the disk, libuv uses a thread pool to provide the illusion that a nonblocking call is being used. Managing a thread pool is no fun at all, but writing email.send('template.ejs', (err, html) => {}) is definitely much easier to understand.

The real benefit to using asynchronous APIs with nonblocking I/O is that Node can do other things while these comparatively slow processes happen. Even though you have only a single-threaded, single-process Node web app running, it can handle more than one connection from potentially thousands of website visitors at any one time. To understand this, you need to look at the **event loop.**

## Event Loop

In any network I/O application, in this case the browser is making http calls to the node application, Node’s built-in HTTP server library, which is a core module called http.Server, handles the request by using a combination of streams, events, and Node’s HTTP request parser, which is native code.

This triggers a callback in your application to run. The callback that runs causes a database query to run, and eventually the application responds with JSON using HTTP. This whole process uses a minimum of three nonblocking network calls: one for the request, one for the database, and another for the response.

How does Node schedule all these nonblocking network operations?

The answer is the event loop. Below figure shows how the event loop is used for these three network operations.

![event-loop](/Users/rishik_rohan/Documents/work/dev/code-academy/nodejs-dev/documentation/assets/event-loop.jpg)

The event loop runs one way (it’s a first-in, first-out queue) and goes through several phases. Figure shows a simplified set of the important phases that run on each iteration of the loop.

- First, the timers execute, which are the timers scheduled with the JavaScript functions setTimeout and setInterval.
- Next, I/O callbacks run, so if any I/O has returned from one of the nonblocking network calls, this is where your callback is triggered.
- The poll phase is where new I/O events are retrieved, and then callbacks scheduled with setImmediate run at the end.This is a special case because it allows you to schedule a callback to run immediately after the current I/O callbacks already in the queue.
