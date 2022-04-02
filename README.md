# multipeer-simple-peer-server

This is a fork of [multipeer-simple-peer-server](https://github.com/vanevery/multipeer-simple-peer-server).

The main reason behind this partial rewrite is that:

1. [`p5LiveMedia`](https://github.com/vanevery/p5LiveMedia) is too high-level and contains some default `p5` operations that is redundant to plain JavaScript projects.
2. The original [`multipeer-simple-peer-server`](https://github.com/vanevery/multipeer-simple-peer-server) uses `Array`s to keep track of peers, which can be easily improved upon using the ES6 `Map` to achieve `O(1)` complexity.
3. The original [`multipeer-simple-peer-server`](https://github.com/vanevery/multipeer-simple-peer-server) does not separate the boilerplate code with user code on the client side, which was a bit inconvenient considering that there were hundreds of lines of code coupled together, and it might be hard to figure out where everything is done and what could be modified and what shouldn’t be.

I abstracted away the boilerplate code into a `MultiPeerConnection` class in [`multipeer-connection.js`](./public/multipeer-connection.js) so that users can easily include that file as a library and establish a multi-peer [`simple-peer`](https://github.com/feross/simple-peer) connection with a [`socket.io`](https://socket.io/) signaling server and customize its behavior without worrying the inner works.

```js
const multiPeerConnection = new MultiPeerConnection({
    host: null, // default to current server if used with `server.js`
    stream: videoStream,
    onStream: receivedStreamCallback,
    onData: receivedDataCallback,
    onPeerDisconnect: peerDisconnectedCallback,
    videoBitrate: 500, // 500kbps
    audioBitrate: 100, // 100kbps
});
```

`MultiPeerConnection` has a `sendData` method that accepts a string and would send that through data channel to all connected peers.

`MultiPeerConnection` also has properties `peers` and `socket` that stores all connected peers and the current user’s [`socket.io`](https://socket.io/) socket, respectively.

The `onStream` callback takes in two parameters, the `MediaStream` object received from [`simple-peer`](https://github.com/feross/simple-peer), and the `SimplePeerWrapper` object from which the media stream is received.

The `onData` callback also takes in two parameters, the data received from [`simple-peer`](https://github.com/feross/simple-peer), and the `SimplePeerWrapper` object from which the data is received.

The `onPeerDisconnect` callback takes just one parameter, the [`socket.io`](https://socket.io/) id of the disconnected peer.

All options and callbacks can be ignored to create a data-channel only connection that can broadcast data but has no callback (which probably isn’t what you want, but that’s legal).
