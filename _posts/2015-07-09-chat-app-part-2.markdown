---
layout: post
title: "Chat App - Part 2: SocketIO"
date: 2015-07-09
categories: javascript
tags: socketio nodejs
excerpt: "Part 2 of a series in building a simple chat application with a pure javascript stack. This post covers using SocketIO for bi-directional communication between client and server."
---
In the previous part, we set up a basic HTTP server that returned a HTML file to
connected clients. We also set up a SocketIO connection between the server and
clients allowing them to communicate with each other using events. Now we will
define our own custom events so that we can send messages between clients for
our chat app.

*Note: I will skip the implementation details of the client facing UI in this
part. I simply used jQuery to grab user input when first prototyping the app.*

#### Events
Events are the backbone of SocketIO's bi-directional communication. We can send
any arbitrary JSON-encodable data between client and server.

To send a message, on the client side we will use an `emit` call after grabbing
the user input.

In `index.html`:

```html
<script type="text/javascript">
    var io = require('socket.io-client');
    var client = io();

    // grab user input
    ...

    // msg is the user input string
    client.emit('send message', msg);

</script>
```

The first argument passed to `emit` is the event name and this can be anything
you want. You should consider coming up with a naming scheme if you plan on
having a large number of events for your app.

The second argument is the data we want to send to the server that this client
is connected to. In the example we are just sending a simple string, but we can
easily send a javascript object for more complex data.

Now we want to listen for this event on the server so that we can receive the
data sent and do something with it. We use the `on` function to achieve this.

In `server.js`:

```javascript
...

io.on('connection', function(socket) {
    ...

    socket.on('send message', function(msg) {
        // do something with msg here
    });
});
```

The first argument passed to `on` is the name of the event we want to listen
for.

The second arguement is the callback that will be executed on receiving an
event. The first parameter of the callback function will be whatever was passed
to the `emit` call for the event in question.

As I previously mentioned, SocketIO is all about bi-directional communication
meaning the server can also `emit` events back to clients. In the case of the
server though, we have several options available to us on who exactly to `emit`
events to.

*Note: We are making these calls within the context of the callback function, so
"sender" here refers to the client that originally fired off the event listener.*

To `emit` an event only to the sender:

```javascript
    ...

    socket.on('send message', function(msg) {
        socket.emit('new message', msg);
    });
```

To `emit` an event to all clients including the sender:

```javascript
    ...

    socket.on('send message', function(msg) {
        io.sockets.emit('new message', msg);
    });
```

To `emit` and event to all clients except the sender:

```javascript
    ...

    socket.on('send message', function(msg) {
        socket.broadcast.emit('new message', msg);
    });
```

The above examples assume that all connected clients are in the default
namespace and room. See the official [documentation](http://socket.io/docs/rooms-and-namespaces/)
for more information, but essentially namespaces and rooms can be used to
implement channels or private chat (something we will add to our app later on).

Our clients also need to be listening for the 'new message' event that the
server emits for them to be able to receive the data so let's add that in.

In `index.html`:

```html
<script type="text/javascript">
    var io = require('socket.io-client');
    var client = io();

    // grab user input
    ...

    // msg is the user input string
    client.emit('send message', msg);

    client.on('new message', function(msg) {
        // display the msg
    });

</script>
```

Depending on what you chose for the server to do, you should now be able to send
messages from one client to another. You can open separate tabs or browsers to
mimic multiple clients. Try sending a message from one client and seeing it
appear in another in real time!

#### Conclusion
We have a working, albeit extremely basic chat app where clients can send each
other messages. In the next part we will work on implementing a proper front-end
UI for our chat app using ReactJS.
