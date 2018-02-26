---
layout: default
title: A TCP relay mechanism with Node.js
tags: nat tcp relay traversal nodejs
---

The TCP relay scripts shown here can be used to expose any TCP/IP service running behind a NAT. This includes services that use HTTP, SSH, and so on. The latest source code is available at [GitHub](https://github.com/tewarid/node-tcp-relay). Use [npm](https://www.npmjs.com/package/node-tcp-relay) to install.

The relay server script is meant to be executed on the server visible on the internet as follows

```bash
tcprelays --relayPort port --servicePort port
```

`relayPort` is the port where the relay server will listen for incoming connections from the relay client. `servicePort` is the port where external clients can connect to the service exposed through the relay.

The relay client script is meant to be executed on a machine behind a NAT as follows

```bash
tcprelayc --host host --port port --relayHost host --relayPort port [--numConn count]
```

`host` is any server visible to the machine behind the NAT, it can also be `localhost`. `port` is the port of the service you want to expose through the relay. `relayServer` is the host or IP address of the server visible on the internet, and already executing the relay server script. `relayPort` is the port where this script will connect with the relay server. `numConn` is the number of unused sockets relay client maintains with the relay server. As soon as it detects data activity on a socket, relay client establishes another connection.

If you're using HTTP, use a reverse proxy such as http-proxy between the relay client and the local service e.g.

```javascript
var httpProxy = require('http-proxy');
httpProxy.createProxyServer({target:'http://host:port'}).listen(port);
```

One external module is used in these scripts to parse commands line options. It can be installed using `npm` as follows

```bash
sudo npm -g install optimist@latest
```

### Programming Interface

Create and start a relay client thus

```javascript
var relayClient = require("node-tcp-relay")
var newRelayClient = relayClient.createRelayClient("hostname", 8080, "relayserver", 10080, 1);
```

End relay client

```javascript
newRelayClient.end();
```

Create and start a relay server thus

```javascript
var relayServer = require("node-tcp-relay");
var newRelayServer = relayServer.createRelayServer(10080, 10081);
```

End relay server

```javascript
newRelayServer.end();
```
