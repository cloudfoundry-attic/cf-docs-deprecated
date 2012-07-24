---
title: Node.js
description: Node.js Application Development with Cloud Foundry
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

Here are steps to create and deploy a "hello world" Node.js web server application that uses the [Express](http://expressjs.com) web module:

### Create the App

Create a directory for the application and change into it.

``` bash
$ mkdir hello-node
$ cd hello-node
```

Use `npm` (Node Package Manager) to install the Express module:

```bash
$ npm install express
```

Create the file `app.js` with the following code:
```javascript

var app = require('express').createServer();
app.get('/', function(req, res) {
    res.send('Hello from Cloud Foundry');
});
app.listen(3000);

```


Create a `package.json` file with the following contents:

```javascript
{
 "name":"hello-node",
  "version":"0.0.1",
  "dependencies":{
      "express":""
  }
}

```

### Deploy the App

Target Cloud Foundry and log in with your Cloud Foundry credentials:

```bash
$ vmc target api.cloudfoundry.com
$ vmc login
```

Push the application. You can press `Enter` to accept the defaults at most of the prompts, but be sure to enter a unique URL for the application. Here is an example push:

``` bash
$ vmc push
	Would you like to deploy from the current directory? [Yn]:
	Application Name: hello-node
	Application Deployed URL [hello-node.cloudfoundry.com]:
	Detected a Node.js Application, is this correct? [Yn]:
	Memory Reservation (64M, 128M, 256M, 512M, 1G) [64M]:
	Creating Application: OK
	Would you like to bind any services to 'hello-node'? [yN]:
	Uploading Application:
	  Checking for available resources: OK
	  Packing application: OK
	  Uploading (0K): OK
	Push Status: OK
	Staging Application: OK
	Starting Application: ................ OK
```

Access the application with your browser at the specified URL, [http://hello-node.cloudfoundry.com](http://hello-node.cloudfoundry.com) in this example.

## Changing the version of Node your App uses

You can use the runtime flag and pass the desired runtime. For how to list the runtimes see the next section

Example to use Node 0.6.8

```bash
vmc push --runtime=node06
```

## Checking the Node.js version on your Cloud Foundry instance

To have the best development experience, we recommend that you work on your local machine with the same version of Node.js as your target Cloud Foundry instance.

To find out what version of Node.js is on your Cloud Foundry instance run this command:

``` bash
$ vmc runtimes

+--------+-------------+-----------+
| Name   | Description | Version   |
+--------+-------------+-----------+
| java   | Java 6      | 1.6       |
| ruby18 | Ruby 1.8    | 1.8.7     |
| ruby19 | Ruby 1.9    | 1.9.2p180 |
| node   | Node.js     | 0.4.12    |
| node06 | Node.js     | 0.6.8     |
| node08 | Node.js     | 0.8.2     |
+--------+-------------+-----------+

```

+ Then download the specific version of Node.js from [here](https://github.com/joyent/node/tags).
+ Unzip it.
+ Install it by following the instructions in the README file.

## Next Steps

+	[Using Cloud Foundry MongoDB services](/services/mongodb/nodejs-mongodb.html) from Node.js applications
+	[Using Cloud Foundry RabbitMQ services](/services/rabbitmq/nodejs-rabbitmq.html) from Node.js
