---
title: Node.js
description: Node.js Collaboration server side
tags:
    - nodejs
    - express
    - tutorial
---

To add collaboration features like broadcasting data received from one client to all other active users add following code in `app.js`.

There are basically 3 methods - onSockjsConnection, broadcast and onDataHandler.

```javascript

sockjs_echo.on('connection', onSockjsConnection);


/* Listen to socket connection and data receive events*/
function onSockjsConnection(conn) {
	clients[conn.id] = conn;
	broadcast({ action: 'newUser' , id: conn.id});
   conn.on('data', function(data) {
			onDataHandler(data, conn.id)
   });
}

/* to broadcast the data received from one client to all other active clients*/
function broadcast (message, exclude) {
	for ( var i in clients ) {
		if ( i != exclude ) clients[i].write( JSON.stringify(message) );
	}
}

/* Handling data received from client to broadcast */
function onDataHandler(data, id) {
	data = JSON.parse(data);
	data.id = id;
	broadcast(data, id);
}

```
The above code is self-explanatory, onSockjsConnection is a call back funciton recieved when 'connection' is made and it returns connection object.<br>
And `broadcast` method has two parameters `message` and `exclude`. Second one is to exclude the client (that sent the data to server) from broadcasting. 
<p><a class="button-plain" style="padding: 3px 15px;" href="/frameworks/nodejs/nodejs-tutorial/step04-nodejs-creatingbasic-view.html">Prev</a>  <a class="button-plain" style="padding: 3px 15px; float: right;" href="/frameworks/nodejs/nodejs-tutorial/step06-collaboration-client-side.html">Next</a></p>