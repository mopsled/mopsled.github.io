---
layout: post
title: "Debugging WebSocket Connections"
description: ""
category:
tags: [websocket, proxy, nodejs]
---

Purpose
---

There are several in-browser and proxy solutions available for viewing and debugging WebSocket connections. This post is about existing WebSocket debugging tools.

I originally looked at these tools for reverse-engineering a client that communicates using the WebSocket protocol. I'll write about the custom NodeJS solution that worked for me in another post.

About WebSockets
---

### Protocol

The [WebSocket](https://en.wikipedia.org/wiki/WebSocket) protocol allows two-way communication between a client and server over a single connection. In standard HTTP, the client must make a request to the server in order to receive new information. WebSockets allow the server to send information to the client without solicitation. In HTTP, each client request results in a new connection to the server. With WebSockets, connections are established once and are reused between requests.

### Existing Proxies

WebSocket and HTTP communication are very different. WebSockets use the HTTP protocol to make an initial handshake with a server, but change to a new protocol once the connection is established. After an HTTP handshake, the client and server communicate over a TCP connection without the use of HTTP.

Because WebSocket connections work differently from standard HTTP connections, existing HTTP debugging proxies (e.g. [Fiddler](http://www.telerik.com/fiddler), [Charles](https://www.charlesproxy.com/)) are generally much less powerful when proxying WebSocket connections.

WebSocket Debugging Choices
---

There are a few existing choices for WebSocket proxying and debugging:

<div id="fiddler"></div>
### Fiddler

[Fiddler](http://www.telerik.com/fiddler) is a powerful web debugging proxy capable of displaying and making changes to WebSocket messages. [Fiddler added the ability to edit WebSocket messages](http://www.telerik.com/blogs/what-s-new-in-fiddler-2-4-4-5) in FiddlerScript using the `FiddlerApplication.OnWebSocketMessage` event. Fiddler is ideal for displaying and debugging WebSocket connections made locally on a Windows client.

- Viewing WebSocket messages:
![Fiddler WebSocket Screenshot](/assets/images/websocket-proxy/fiddler-websocket.png)

- This is an example Fiddler script for changing WebSocket message payloads. This `CustomRules.cs` FiddlerScript file changes WebSocket messages from

<pre><code>"Hello, &lt;something&gt;!"</pre></code>

to

<pre><code>"Hello, FORGED-&lt;something&gt;!"</pre></code>


`CustomRules.cs`:

      class Handlers
      {
          // ...

          static function OnWebSocketMessage(oMsg: WebSocketMessage)
          {
              // Modify a message's content
              var sPayload = oMsg.PayloadAsString();
              var pattern = "Hello, \([a-zA-Z]+\)!";
              var match = Regex.Match(sPayload, pattern);

              if (match.Success) {
                  var pattern = "Hello, \([a-zA-Z]+\)!";
                  var match = Regex.Match(sPayload, pattern);
                  var who = match.Groups[1].ToString();

                  var forgedWho = String.Format("FORGED-{0}", who);
                  var changedPayload = sPayload.Replace(who, forgedWho);
                  FiddlerApplication.Log.LogString(String.Format("Changing {0} to {1}", who, forgedWho));
                  oMsg.SetPayload(changedPayload);
              }
          }
      }

Running this script in conjunction with [Kaazing's WebSocket echo test](kaazing.org/demos/echo/run):
![Kaazing echo with forged message](/assets/images/websocket-proxy/fiddler-edit.png)

### Charles

The HTTP proxy [Charles](https://www.charlesproxy.com/) added the ability to [view WebSocket connections in 2015](https://www.charlesproxy.com/documentation/version-history/). Charles displays an iMessage-style view of a WebSocket conversation. At the time of writing, Charles does not provide any way to change or further debug WebSocket messages.

![Charles WebSocket Screenshot](/assets/images/websocket-proxy/charles-websocket.png)

### Chrome

WebSocket connections in Chrome can be viewed by clicking on a WebSocket connection in the Network Tab of Chrome's developer tools:

![Chrome view of WebSockets](/assets/images/websocket-proxy/chrome-websocket.png)

Similar to Charles, Chrome can only view existing connections and doesn't have the ability to script or modify WebSocket connections.

### [node-http-proxy](https://github.com/nodejitsu/node-http-proxy)

This NodeJS package is a library for proxying and modifying HTTP requests. However, at the time of writing, `node-http-proxy`'s WebSocket proxying capability does not allow the modification of individual messages. Here's an example that proxies `ws://localhost:10000` to `wss://echo.websocket.org`, and prints messages received :

`http-proxy-websocket-print.js`

    var httpProxy = require('http-proxy');
    var http = require('http');

    var proxy = httpProxy.createProxyServer({});
    proxy.on('open', function (proxySocket) {
     proxySocket.on('data', function (data) {
       console.log('message from echo.websocket.org: %s', data.toString('utf8'));
     });
    });

    var server = http.createServer();
    server.on('upgrade', function (req, socket, head) {
      console.log("Proxying websocket connection to wss://echo.websocket.org");
      proxy.ws(req, socket, head, {
       target: "wss://echo.websocket.org",
       changeOrigin: true,
       ws: true});
    });

    server.listen(10000);

Then, connect to `ws://localhost:10000`, which forwards the connection to `wss://echo.websocket.org`:

![WebSocket to local proxy](/assets/images/websocket-proxy/nodejs-proxy-web.png)

Example output:

{% highlight bash %}
Proxying websocket request to wss://echo.websocket.org
message from echo.websocket.org: Hello, WebSocket!
{% endhighlight %}

Selection
---

For viewing WebSocket connections, Chrome and Charles are adequate choices. However, [Fiddler's scripting capabilities](#fiddler) provide the most power for changing message payloads on-the-fly with regular expressions and C#.

For my own project, I chose to use a small wrapper around [nodejs-websocket](https://github.com/sitegui/nodejs-websocket) to have more control over request headers, proxying HTTP->HTTPS WebSocket connections, and payload scripting capability in NodeJS. I'll discuss this in my next post.
