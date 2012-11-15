---
title: Node.js
description: Developing Whiteboard app
tags:
    - nodejs
    - express
    - tutorial
---

This is a guide for Node.js developers deploying on Cloud Foundry. It shows you how to set up and successfully deploy Node.js applications to Cloud Foundry.

Node.js is an event-driven, scalable, JavaScript-based platform for networking applications. Cloud Foundry provides a runtime environment for Node.js applications and the Cloud Foundry deployment tools automatically recognize Node.js applications.

Before you get started, you need the following:

+	A [Cloud Foundry account](http://cloudfoundry.com/signup)

+	The [vmc](/tools/vmc/installing-vmc.html) Cloud Foundry command line tool

+	A [Node.js](http://nodejs.org/) installation matching the version of Node.js on your Cloud Foundry instance. See [Checking the NodeJS version on your Cloud Foundry instance](#checking-the-nodejs-version-on-your-cloud-foundry-instance) below.

## Deploying Node.js Applications to Cloud Foundry

When you deploy a Node.js application to Cloud Foundry, the current directory must contain the application, `app.js`, and, if your application depends on any modules, a `package.json` file that names them.

Here are steps to create and deploy a "Whiteboard" Node.js web server application that uses the [Express](http://expressjs.com) web module, [Sockjs](https://github.com/sockjs/sockjs-node) and [Fabric.js](http://fabricjs.com/):

### Create the App

Create a directory 'nodejs-whiteboard' for the application and change into it.

``` bash
$ mkdir nodejs-whiteboard
$ cd nodejs-whiteboard
```

Use `npm` (Node Package Manager) to install the Express module and Sockjs:

```bash
$ npm install express
$ npm install sockjs
```

Create the file `app.js` with the following code:
```javascript

var express = require('express');
var sockjs = require('sockjs');
var http = require('http');
var app = express();
var clients = {};

var sockjs_opts = {sockjs_url: "http://cdn.sockjs.org/sockjs-0.3.min.js", websocket:false};
var sockjs_echo = sockjs.createServer(sockjs_opts);
var server = http.createServer(app);

app.use('/static', express.static(__dirname + '/static'));
sockjs_echo.installHandlers(server, {prefix:'/echo'});
server.listen(process.env.VCAP_APP_PORT || 4000);

app.get('/', function (req, res) {
    res.sendfile(__dirname + '/index.html');
});

```

Create a `package.json` file with the following contents:

```javascript
{
 "name":"nodejs-whiteboard",
  "version":"0.0.1",
  "dependencies":{
      "express":"",
      "sockjs":""
  }
}

```

Now create a `index.html` page for our whiteboard app with the following markups:

```html
<!doctype html>
<html lang="en">
   <head> </head>
   <body lang="en">
      Welcome to whiteboard app
   </body>
</html>

``` 
We are done with basic code. To start the server run following command:

```bash
$ node app.js

```
 
## Check point
Open your browser and type http://localhost:4000. You will see a page with 'Welcome to whiteboard app' text

![Welcome screen](/images/screenshots/nodejs-whiteboard/welcome.png)

<p><a class="button-plain"  style="padding: 3px 15px;" href="/frameworks/nodejs/nodejs-tutorial/step02-technology-used.html">Prev</a>  <a class="button-plain"  style="padding: 3px 15px; float: right;" href="/frameworks/nodejs/nodejs-tutorial/step04-nodejs-creatingbasic-view.html">Next</a></p>

