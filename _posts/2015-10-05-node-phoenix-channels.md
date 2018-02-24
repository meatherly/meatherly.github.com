---
layout: post
title: "Node and Phoenix Channels"
description: "Simple post on connecting Node app to your phoenix channels."
category:
tags: [js node phoenix]
---


I've been working with a Raspi lately and needed to connect it to my Phoenix app. After talking with [chrismccord](https://github.com/chrismccord) he showed me it was easier than I thought. See below.

Things you'll need:
* [websocket](https://www.npmjs.com/package/websocket) - For websocket transport.
* [Phoenix.JS](https://github.com/phoenixframework/phoenix/blob/master/web/static/js/phoenix.js)
* [babel](https://babeljs.io/) - For transpiling the ES6 phoenix.js file.


Once you have everything above you can just throw it in your node app just like so:

```
import {Socket} from "./phoenix.js"
var socket = new Socket("ws://127.0.0.1:4000/socket", {transport: require('websocket').w3cwebsocket})

socket.connect();
let channel = socket.channel("raspis:lobby");

channel.join()
  .receive("ok", resp => {
    console.log("Boom!")
  })
  .receive("error", reason => console.log(`Error joining channel: ${reason}`))
```

To see how to use the Phoenix.JS library head over to http://www.phoenixframework.org/docs/channels

That's about it. Simple right?
