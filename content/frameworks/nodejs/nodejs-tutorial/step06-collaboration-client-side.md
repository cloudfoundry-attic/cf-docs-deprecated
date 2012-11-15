---
title: Node.js
description: Node.js Collaboration client side
tags:
    - nodejs
    - express
    - tutorial
---

To add collaboration features on client side, we need to add sockjs listener code.

Below is the code snippet from `main.js`.

Create an instance of sockjs with the following line of code.


```javascript
	this.sockjs_url = '/echo';
	this.sockjs = new SockJS(this.sockjs_url);

```
Here `this` refers to `app` name space we have defined in `main.js`.

Now lets assign methods for sockjs events. 

```javascript

	this.sockjs.onopen = this.onSocketOpen;
	this.sockjs.onmessage = this.onSocketMessage;	
	this.sockjs.onclose = this.onSocketClose;
		
```
Below is the method declaration for above three methods.

onSocketOpn method
```javascript
onSocketOpen: function(){
	$('#spinner').hide(); 
	$('#wait').hide();
	app.displayMessage('[*] open', app.sockjs.protocol);
}	
		
```
<br>
onSocketMessage
```javascript
onSocketMessage: function(e) {
	var data = JSON.parse(e.data);
	switch ( data.action ) {
		case 'newUser':
			app.userJoined(data);
		break;
		case 'text':
			app.textMessage(data);
		break;
		case 'new_shape':
			app.createNewShape(data);
		break;
		case 'modified':
			app.modifyObject(data);
		default:
			//
		break;
	}
}
```
<br>
onSocketClose
```javascript
onSocketClose: function(){
	app.displayMessage('[*] close', '');
},	
```

Let's take a look at the individual pieces of the snippet above. The argument provided to SockJs represents the URL to the address listening for socket connections.<br> 
`onopen`, `onclose`, and `onmessage` methods connect you to events on the socket instance. Each of these methods provides an event which gives insight as to the state of the socket.<br>
The `onmessage` event provides a data property which contains the body of the message. The message body must be a string, so serialization/deserialization strategies will be needed to pass more data.<br>

`onSocketMessage` assumes four types of data from server:

 * 'newUser' - when user joins the app intimating the same to Server to broadcast it to all clients.
 * 'text' - when user types a text in input box.
 * 'neew_shape' - when  user creates a new shape on cavnas.
 * 'modified' - when user modifies a shape he created on canvas.

We will discuss more about these methods later.

<p><a class="button-plain" style="padding: 3px 15px;" href="/frameworks/nodejs/nodejs-tutorial/step05-collaboration-server-side.html">Prev</a>  <a class="button-plain" style="padding: 3px 15px; float: right;" href="/frameworks/nodejs/nodejs-tutorial/step07-defining-basicshapes.html">Next</a></p>