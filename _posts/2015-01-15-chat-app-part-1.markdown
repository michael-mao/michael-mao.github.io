---
layout: post
title: "Chat App - Part 1: Setup"
date: 2015-05-02
categories: javascript
tags: socketio nodejs reactjs
excerpt: "Part 1 of a series in building a simple chat application with a pure javascript stack. We will use SocketIO, NodeJS, and ReactJS. This post will cover the initial setup of the project."
---
We will build a simple chat app using the following javascript libraries:

- [SocketIO](http://socket.io/)
- [NodeJS](https://nodejs.org/)
- [ReactJS](https://facebook.github.io/react/)

This series will provide a high-level overview of building the chat app with key
code snippets. More in-depth explanations of library specific concepts and features
are deferred to the extensive and well written documentation of each library.

The latest code for this project can be found on GitHub [here](https://github.com/michael-mao/chat-app).
Note that we are starting the project from scratch, therefore the code shown below
may not match the repository code exactly.

#### Project Structure

To begin, we will first setup the project structure as follows:

```
app/
  - models.js
  - utils.js
client/
  - css/
    - main.css
  - img/
  - js/
    - client.js
  - index.html
test/
config.js
package.json
server.js
```

The `app/` directory will contain any helper files needed by the server such as model
definitions and utility functions. The `client/` directory will contain all the front-end
files of our chat application. These will be the files that are served to any clients
that connect to the server. Finally, the `test/` directory will hold all the unit
tests for the server.

#### Dependencies

Every node project requires a `package.json` file, so run `npm init` in the project
directory if you do not have already have one. Then add the following dependencies
to `package.json`:

```javascript
"dependencies": {
    "express": "4.10.2",
    "socket.io": "1.3.2",
    "react": "0.12.2",
    "jquery": "1.11.2",
    "bootstrap": "3.3.2",
    "socket.io-client": "1.3.3"
}
```

Run `npm intall` in the project directory to install the newly listed dependencies.

#### Server

Now that we have the dependencies we need, let's setup a simple HTTP server that
can listen for requests and serve a static file.

 In `server.js`:

```javascript
'use strict';

// Vendor packages
var express = require('express');
var http = require('http');
var socketio = require('socket.io');

// Setup server
var app = express();
var server = http.Server(app);
var io = socketio(server);

app.use(express.static(__dirname + '/client'));
app.get('/', function(req, res) {
    res.sendFile(__dirname + '/client/index.html');
});

server.listen(8080, function() {
    console.log('server listening on port 8080');
});
```

In `index.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Chat App</title>
</head>
<body>
    <p>Hello World!</p>
</body>
</html>
```

The above code sets up an [Express](http://expressjs.com/) app that returns `index.html`
when visiting the root domain (`/`) of the server. The server is then configured
to listen for requests on port `8080` of the local machine.

Run `node server.js` and we should now be able to visit `http://localhost:8080` in
a browser and see the contents of `index.html`.

#### SocketIO

We now have an operational server, but it only serves static content. To enable
bi-directional communication between server and client, we'll use SocketIO.

In `server.js` append the following:

```javascript
...

io.on('connection', function(socket) {
    console.log('Client connection: ' + socket.id);
});
```

In `index.html` replace the `<body>` contents with:

```html
<script type="text/javascript">
    // Vendor packages
    var io = require('socket.io-client');

    // Setup socket.io connection
    var client = io();
</script>
```

SocketIO communicates by sending and receiving events. The `connection` event is
automatically fired when a new socket connects to the server and is one of SocketIO's
predefined events. However, you can send and receive any custom event that you define,
along with any JSON encoded data you want.

The above code will now log the unique socket id of any client that connects to the
server. Note that we can simply call `io()` in the client because it will default
to connecting to the host server.

#### Conclusion

Our server and clients will now be able to talk to each other using SocketIO's event-based
communication. It may not look like much, but we have set up the foundation for our
chat app. In the next part we will begin defining our own custom events to send data
back and forth, such that clients will be able to send and receieve messages through
the server.

