---
title: Node.js
description: New Whiteboard app with SockJS and FabricJS
tags:
    - nodejs
    - express
    - tutorial
---

## Introduction

This section introduces the Whiteboard application, teaches you how to get started with a new Node.js application with Sockjs and touches upon some other configurations required for the application.

## Prerequisites

+ [Node.js](http://howtonode.org/how-to-install-nodejs) is installed on your system to build node application.
+ [Sockjs](https://github.com/sockjs/sockjs-node) server is installed on you system.
+ [Express](http://expressjs.com/) a web application framework for node, is also installed on your system.
+ You are proficient in developing node.js application.


## Subtopics
+ [Whiteboard Application Overview](#wbappoverview)
+ [Server Module](#servermodule)
+ [SockJS Events](#sockjsevents)


## Whiteboard Application Overview <a id="wbappoverview"/>

This application is a realtime collaborative White board application that uses a canvas to draw shapes. 

#### Create the App

Create a directory 'nodejs-whiteboard' for the application and change into it.

``` bash
$ mkdir nodejs-whiteboard
$ cd nodejs-whiteboard
```

Use `npm` (Node Package Manager) to install the Express, static modules and Sockjs server:

```bash
$ npm install express
$ npm install sockjs
$ npm install static
```
## Server Module<a id="servermodule"/>

Create the file app.js in the root directory of your project, and fill it with the following code:

```javascript
//app.js
(function () {
  /**
   * Initialize app by importing required modules and starting servers
   * @method init
   * @param none
   */
  function init() {
    var express = require('express'),
        http = require('http'),
        app = express();

  /* Set static folder */
  app.use('/static', express.static(__dirname + '/static'));

  /* Redirect user to index.html when requested for root */
  app.get('/', function (req, res) {
    res.sendfile(__dirname + '/index.html');
  });

  loadConfig(function(dataObj) {
     /* Create http server */
     var server = http.createServer(app);
     
     /* Server is set to listent to specific port */
     server.listen(process.env.VCAP_APP_PORT || dataObj.port);
     
     /* Initialize the SockJS server*/
     initSockServer(server);
   });
 }

 /**
  * Loads configuration.json file and sends data to callback
  * @method loadConfig
  * @param {Function} callback
  */

 function loadConfig(callback) {
   var fs = require('fs');
   /* Load configuration data */
   fs.readFile(__dirname + '/config.json', function(err, data) {
      callback(JSON.parse(data))
   });
 }

 /**
  * Initializes the sockJS server
  * @method initSockServer
  * @param  server - http server instance
  */

 function initSockServer(server) {
   var sockJSServer = require('./sockJSServer');
   sockJSServer.initSockJS(server);
 }
 init();
})();

```

The above file loads `config.json` and `sockJSServer.js`

Create a file called `config.json` in the root directory of your project and add the below code.

``` javascript
//config.json
 {
   "port" : "4000"	
 }

```

In the Cloud Foundry environment, you will be using the port given to you as a part of the environment variables, VMC_APP_PORT, rather than any hard coded port number. Find the code for it here.

``` javascript 
  //app.js
  server.listen(process.env.VCAP_APP_PORT || dataObj.port);
```

Create a file called `sockJSServer.js` in the root directory of your project. Below is the code to be written in `sockJSServer.js` to start sockJS server.
This creates a sockJS server to create a communication channel between the browser and the web server.

```javascript
//sockJSServer.js
var sockJSServer = {
 initSockJS:function (server) {
     var sockjs = require('sockjs');

     /* Create sockjs server */
     var sockjs_echo = sockjs.createServer({
         sockjs_url:"http://cdn.sockjs.org/sockjs-0.3.min.js",
         websocket:false
     });

     /* We call installHandlers passing in the app server so we can listen and answer any incoming requests on the echo path.*/
     sockjs_echo.installHandlers(server, {
         prefix:'/echo'
   });
 }
}	  
     /* make it as a node module */ 	     
     module.exports = sockJSServer;
     
```  
Create a `package.json` file with the following contents:

```javascript
//package.json
{
 "name":"nodejs-whiteboard",
  "version":"0.0.1",
  "dependencies":{
      "express":"",
      "sockjs":""
  }
}

```
Now create a `index.html` page for our whiteboard app with the following markups and save it in the root:

```html
<!--index.html-->
<!doctype html>
<html lang="en">
   <head> </head>
   <body lang="en">
      Welcome to whiteboard app
   </body>
</html>

``` 
That's it! You just wrote a working HTTP and SockJs server. Verify it by running and testing it. First, execute your script with Node.js: 
Starting a new application in Node.js is as simple as executing a single command 'node <appname>'.

```bash
$ node app.js

```
 
## Check point
Open your browser and type http://localhost:4000. You will see a page that says 'Welcome to Whiteboard app'.

![Welcome screen](/images/screenshots/nodejs-whiteboard/welcome.png)

## SockJS Events<a id="sockjsevents"/>

Now write event listener functions.

On the server side you need to have two methods for the collaboration feature.

+ Listen for connection event
+ Listen for data events on the client

Add the below code in `sockJSServer.js' file. 

#### Note: Code should be inserted inside main object of the file mentioned. 
Here it is `var sockJSServer = { existing code + //your code goes here }`


```javascript
//sockJSServer.js
/* List of incoming clients will be maintained in this map */
	var clients = {};
	var clientData = {};

/* Listens for incoming connect events */
	sockjs_echo.on('connection', onSockJSConnection);
	
/* Is triggered when a new user joins in */
  function onSockJSConnection(conn) {
       /* Add him to the clients list */
      clients[conn.id] = conn;

      /* Send new users all the previous data to update their whitebaord */
      sendShapeActionsToClient(conn, clientData);
      sendChatDataToClient(conn, clientData);

      /* Listen for data events on this client */
      conn.on('data', function (data) {
          onDataHandler(data, conn.id);
          pushUserData(data);
      });

      conn.on('close', function (data) {
          delete clients[conn.id];
          onDataHandler(JSON.stringify({userName: conn.userName, action:'text', message: 'left'}), conn.id);
      });
  }
```
Whenever a new client joins, you need to push the connection object to clients list to maintain a list of clients.

```javascript
   //sockJSServer.js
	clients[conn.id] = conn;
```
### Broadcasting data

The data that server receives from client needs to be broadcasted to all other clients.
```javascript
//sockJSServer.js
onDataHandler(data, conn.id);

```
Below is the implementation of the above method

```javascript
//sockJSServer.js
/* Handling data received from client to broadcast */
  function onDataHandler(dataStr, id) {
      var dataObj = JSON.parse(dataStr);
      dataObj.id = id;
      broadcast(dataObj);
  }

 /* To broadcast the data received from one client to all other active clients */
  function broadcast(message, includeSelf) {
      for (var index in clients) {
          if (clients.hasOwnProperty(index)) {
              var client = clients[index];
              if (includeSelf || client.id !== message.id) {
                  client.write(JSON.stringify(message));
              }
          }
      }
  }
```
Every new user who joins the board needs to be shown all previous drawings. In order to do that, the data from all clients need to be captured and then sent. To do so, you  write two methods.

```javascript
//sockJSServer.js
/* Send new users all the previous data to update their whitebaord */
   sendShapeActionsToClient(conn, clientData);
   sendChatDataToClient(conn, clientData);
   
```
Below is the code for pushing all data from clients.

```javascript
//sockJSServer.js
  function pushUserData(data) {
      /* Push the data received from client*/
      var dataObj = JSON.parse(data);
      if (dataObj.action !== 'text') {
          pushShapesData(dataObj);
      } else {
          pushChatData(dataObj);
      }

  }

  function pushChatData(dataObj)  {
      if (clientData.textObj === undefined) {
          clientData.textObj = [];
      }
      clientData.textObj.push(dataObj);
  }

  function pushShapesData(dataObj) {
      var shapeId = dataObj.args[0].uid;
      switch (dataObj.action) {
          case 'new_shape':
              clientData[shapeId] = dataObj;
              break;
          case 'modified':
              if (clientData[shapeId] !== undefined && clientData[shapeId].modify === undefined) {
                  clientData[shapeId].modify = {};
              }
              clientData[shapeId].modify[shapeId] = dataObj;
              break;
          case 'deleted':
              delete clientData[shapeId];
              break;
      }
  }

```
To send data to new users, add the below methods.


```javascript
//sockJSServer.js
function sendShapeActionsToClient(connection, shapesData) {
      for (var index in shapesData) {
          if (shapesData.hasOwnProperty(index)) {
              if (shapesData[index].action === 'new_shape') {
                  connection.write(JSON.stringify(shapesData[index]));
              }
              if (shapesData[index].modify) {
                  for (var action in shapesData[index].modify) {
                      connection.write(JSON.stringify(shapesData[index].modify[action]));
                  }
              }
          }
      }
  }

  function sendChatDataToClient(connection, chatData) {
      if (chatData.textObj) {
          for (var action in chatData.textObj) {
              connection.write(JSON.stringify(chatData.textObj[action]));
          }
      }
  }

```

<p><a class="button-plain"  style="padding: 3px 15px;" href="/frameworks/nodejs/nodejs-tutorial/nodejs-app-with-sockjs.html">Prev</a>  <a class="button-plain"  style="padding: 3px 15px; float: right;" href="/frameworks/nodejs/nodejs-tutorial/step02-creating-ui.html">Next</a></p>

