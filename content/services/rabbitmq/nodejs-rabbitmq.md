---
title: Using RabbitMQ with Node.js
description: Node.js Application Development with RabbitMQ Service
tags:
    - nodejs
    - rabbitmq
    - tutorial
---

Node.js is an event-driven, scalable, Javascript-based platform for networking applications. Cloud Foundry provides a runtime environment for Node.js applications and the Cloud Foundry deployment tools recognize Node.js applications.

RabbitMQ is a message broker that provides robust messaging services for your applications.

This is a guide for Node.js developers who are using the Cloud Foundry RabbitMQ service. It shows you how to access the Cloud Foundry RabbitMQ services in your applications.

See [Node.js Application Development with Cloud Foundry](/frameworks/nodejs/nodejs.html) for a first tutorial on deploying Node.js applications.

### Setup

Confirm Node.js is correctly installed:

Run node to start the interactive JavaScript console. Press `Control-C` twice to exit.

```bash
$ node
```

Check that Node Package Manager (NPM) is installed. At the command line:

```bash
$ npm -v
1.0.6
```

Target Cloud Foundry and log in with your Cloud Foundry credentials:

```bash
$ vmc target api.cloudfoundry.com
$ vmc login
```

### Create Application Files

Create an application directory and change into it:

```bash
$ mkdir rabbitmq-node
$ cd rabbitmq-node
```

Create the file `package.json` with the following contents:

``` javascript
{
    "name":"node-amqp-demo",
    "version":"0.0.1",
    "dependencies": {
        "amqp":">= 0.1.0",
        "sanitizer": "*"
    }
}
```

This `package.json` specifies two dependent libraries: amqp for message processing and sanitizer to escape HTML messages.

Install the dependencies, using npm (Node Package Manager):

```bash
$ npm install
```

Create `app.js` with the following code:

``` javascript
var http = require('http');
var amqp = require('amqp');
var URL = require('url');
var htmlEscape = require('sanitizer/sanitizer').escape;

function rabbitUrl() {
  if (process.env.VCAP_SERVICES) {
    conf = JSON.parse(process.env.VCAP_SERVICES);
    return conf['rabbitmq-2.4'][0].credentials.url;
  }
  else {
    return "amqp://localhost";
  }
}

var port = process.env.VCAP_APP_PORT || 3000;

var messages = [];

function setup() {

  var exchange = conn.exchange('cf-demo', {'type': 'fanout', durable: false}, function() {

    var queue = conn.queue('', {durable: false, exclusive: true},
    function() {
      queue.subscribe(function(msg) {
        messages.push(htmlEscape(msg.body));
        if (messages.length > 10) {
          messages.shift();
        }
      });
      queue.bind(exchange.name, '');
    });
    queue.on('queueBindOk', function() { httpServer(exchange); });
  });
}

function httpServer(exchange) {
  var serv = http.createServer(function(req, res) {
    var url = URL.parse(req.url);
    if (req.method == 'GET' && url.pathname == '/env') {
      printEnv(res);
    }
    else if (req.method == 'GET' && url.pathname == '/') {
      res.statusCode = 200;
      openHtml(res);
      writeForm(res);
      writeMessages(res);
      closeHtml(res);
    }
    else if (req.method == 'POST' && url.pathname == '/') {
      chunks = '';
      req.on('data', function(chunk) { chunks += chunk; });
      req.on('end', function() {
        msg = unescapeFormData(chunks.split('=')[1]);
        exchange.publish('', {body: msg});
        res.statusCode = 303;
        res.setHeader('Location', '/');
        res.end();
      });
    }
    else {
      res.statusCode = 404;
      res.end("This is not the page you were looking for.");
    }
  });
  serv.listen(port);
}

console.log("Starting ... AMQP URL: " + rabbitUrl());
var conn = amqp.createConnection({url: rabbitUrl()});
conn.on('ready', setup);

// ---- helpers

function openHtml(res) {
  res.write("<html><head><title>Node.js / RabbitMQ demo</title></head><body>");
}

function closeHtml(res) {
  res.end("</body></html>");
}

function writeMessages(res) {
  res.write('<h2>Messages</h2>');
  res.write('<ol>');
  for (i in messages) {
    res.write('<li>' + messages[i] + '</li>');
  }
  res.write('</ol>');
}

function writeForm(res) {
  res.write('<form method="post">');
  res.write('<input name="data"/><input type="submit"/>');
  res.write('</form>');
}

function printEnv(res) {
  res.statusCode = 200;
  openHtml(res);
  for (entry in process.env) {
    res.write(entry + "=" + process.env[entry] + "<br/>");
  }
  closeHtml(res);
}

function unescapeFormData(msg) {
  return unescape(msg.replace('+', ' '));
}
```

Push the application to Cloud Foundry. For most prompts you can press `Enter` to accept the default value.
At the Application Name prompt, enter a unique application name. Also, be sure to create and bind a rabbitmq service, as shown in the following transcript.

```bash
$ vmc push
	Would you like to deploy from the current directory? [Yn]:
	Application Name: rabbitmq-node
	Application Deployed URL [rabbitmq-node.cloudfoundry.com]:
	Detected a Node.js Application, is this correct? [Yn]:
	Memory Reservation (64M, 128M, 256M, 512M) [64M]:
	Creating Application: OK
	Would you like to bind any services to 'rabbitmq-node'? [yN]: y
	Would you like to use an existing provisioned service? [yN]: n
	The following system services are available
	1: mongodb
	2: mysql
	3: postgresql
	4: rabbitmq
	5: redis
	Please select one you wish to provision: 4
	Specify the name of the service [rabbitmq-6c981]:
	Creating Service: OK
	Binding Service [rabbitmq-6c981]: OK
	Uploading Application:
	  Checking for available resources: OK
	  Processing resources: OK
	  Packing application: OK
	  Uploading (1K): OK
	Push Status: OK
	Staging Application: OK
	Starting Application: OK

```

Access the application at the specified URL using your browser. Enter messages in the form and see them echoed in the browser.

![RabbitMQ with Node.js application](/images/screenshots/nodejs-rabbitmq/rmq-app.png)
