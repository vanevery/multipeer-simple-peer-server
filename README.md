# MultiPeerConnection

**A More Efficient and User-Friendly [`multipeer-simple-peer-server`](https://github.com/vanevery/multipeer-simple-peer-server)**

The main reason behind this partial rewrite is that:

1. [`p5LiveMedia`](https://github.com/vanevery/p5LiveMedia) is too high-level and contains some default `p5` operations that is redundant to plain JavaScript projects.
2. [`multipeer-simple-peer-server`](https://github.com/vanevery/multipeer-simple-peer-server) uses `Array`s to keep track of peers, which can be easily improved upon using the ES6 `Map` to achieve `O(1)` complexity on the client side, and is actually not necessary at all on the server side since `socket.io` already exposes all socket ids with [`io.adapter.sids`](https://socket.io/docs/v4/rooms/#implementation-details) and we can emit events to a specific client with `io.to(SOCKET_ID).emit` since `socket.io` [automatically puts each client to a room with their own id](https://socket.io/docs/v4/rooms/#default-room) by default.
3. [`multipeer-simple-peer-server`](https://github.com/vanevery/multipeer-simple-peer-server) does not separate the boilerplate code with user code on the client side, which was a bit inconvenient considering that there were hundreds of lines of code coupled together, and it might be hard to figure out where everything is done and what could be modified and what shouldn’t be.

I abstracted away the boilerplate code into a `MultiPeerConnection` class in [`multi-peer-connection.js`](./public/multi-peer-connection.js) so that users can easily include that file as a library and establish a multi-peer [`simple-peer`](https://github.com/feross/simple-peer) connection with a [`socket.io`](https://socket.io/) signaling server and customize its behavior without worrying the inner works.

```js
const multiPeerConnection = new MultiPeerConnection({
    socket: io.connect(host), // could either provide a socket instance or specify a host name
    // host: null, // default to current server if used with `server.js
    streams: new Set([videoStream]), // a Set of all available streams
    onStream: receivedStreamCallback,
    onData: receivedDataCallback,
    onPeerConnect: peerConnectedCallback,
    onPeerDisconnect: peerDisconnectedCallback,
    videoBitrate: 500, // 500kbps
    audioBitrate: 100, // 100kbps
});
```

`MultiPeerConnection` has a `sendData` method that accepts a string and would send that through data channel to all connected peers, an `addStream` and a `removeStream` method that accept a `MediaStream` object and add or remove the stream for all the peers, a `removeStreamsTo` method that accepts the id of a connected peer and remove all streams sent to that peer, and a `close` method that closes the peer-to-peer connection with all peers.

`MultiPeerConnection` also has properties `peers`, `socket`, and `streams` that stores all connected peers, the current user’s [`socket.io`](https://socket.io/) socket, and all the `MediaStream` objects being sent to the peers, respectively.

The `onStream` callback takes in two parameters, the `MediaStream` object received from [`simple-peer`](https://github.com/feross/simple-peer), and the `SimplePeerWrapper` object from which the media stream is received.

The `onData` callback also takes in two parameters, the data received from [`simple-peer`](https://github.com/feross/simple-peer), and the `SimplePeerWrapper` object from which the data is received.

The `onPeerConnect` and `onPeerDisconnect` callbacks take just one parameter, the [`socket.io`](https://socket.io/) id of the connected or disconnected peer.

All options and callbacks can be ignored to create a data-channel only connection that can broadcast data but has no callback (which probably isn’t what you want, but that’s legal).
