---
title: Using MongoDB with Node.js
description: Node.js Development with the MongoDB Service
tags:
    - mongodb
    - nodejs
---

Before you begin, review these prerequisites:

+	[Cloud Foundry account](http://cloudfoundry.com/signup) and [vmc](/tools/vmc/installing-vmc.html)

+	[Node.js](http://nodejs.org/) installed on your development machine

+	[MongoDB](http://www.mongodb.org), the open source NoSQL database, installed on your development computer

See [Node.js Application Development with Cloud Foundry](/frameworks/nodejs/nodejs.html) for a first tutorial on deploying Node.js applications.

## Using the Cloud Foundry MongoDB Service

MongoDB is a scalable, open source, document-oriented database provided as a service at Cloud Foundry. This section describes how you can create Node.js applications that use this service. The examples are coded so that you can run and test them locally as well as on Cloud Foundry.
Code for these examples can be found at [GitHub](https://github.com/gatesvp/cloudfoundry_node_mongodb).

### Setup

Start `mongod` in your local environment with the following command:

``` bash
$ mongod
```

Confirm that Node.js is correctly installed:

+	Run `node` to start the interactive javascript console:

```bash
$ node
```

Press `Control-C` to exit.

+	Check that Node Package Manager (NPM) is installed:

```bash
$ npm -v
1.0.6
```

Target Cloud Foundry and log in with your Cloud Foundry credentials:

``` bash
$ vmc target api.cloudfoundry.com
$ vmc login
```

### Create a Working Node.js Application for Cloud Foundry

In this step, you create a basic application and make sure that it works locally and on Cloud Foundry.

Note
: Throughout this tutorial, we use the name "mongo-node" for the application. When you deploy to Cloud Foundry you must choose a different, unique name.

Create an application directory and change into it:

``` bash
$ mkdir mongo-node
$ cd mongo-node
```

Create a file `app.js` containing the following code:

``` javascript
var port = (process.env.VMC_APP_PORT || 3000);
var host = (process.env.VCAP_APP_HOST || 'localhost');
var http = require('http');

http.createServer(function (req, res) {
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('Hello World\n');
}).listen(port, host);
```

This creates a Node.js web server using port 3000 on localhost that responds to any HTTP request with the string "Hello World."

Start the Node.js web server locally:

``` bash
$ node app.js
```

In another terminal window send a request:

``` bash
$ curl localhost:3000
Hello World
```

Alternatively, browse to http://localhost:3000 to see the response from the web server.

Press `Control-C` in the first terminal window to stop the web server.

Push the application to Cloud Foundry. Press `Enter` to accept the defaults, except enter a unique name for the application, and set up a mongodb service, as shown in the following vmc transcript:

```bash
$ vmc push
    Would you like to deploy from the current directory? [Yn]:
    Application Name: mongo-node
    Application Deployed URL [mongo-node.cloudfoundry.com]:
    Detected a Node.js Application, is this correct? [Yn]:
    Memory Reservation (64M, 128M, 256M, 512M, 1G) [64M]:
    Creating Application: OK
    Would you like to bind any services to 'mongo-node'? [yN]: y
    The following system services are available
    1: mongodb
    2: mysql
    3: postgresql
    4: rabbitmq
    5: redis
    Please select one you wish to provision: 1
    Specify the name of the service [mongodb-e840e]:
    Creating Service: OK
    Binding Service [mongodb-e840e]: OK
    Uploading Application:
      Checking for available resources: OK
      Packing application: OK
      Uploading (0K): OK
    Push Status: OK
    Staging Application: OK
    Starting Application: OK
```

Cloud Foundry stages and starts your application.

Test the application.

``` bash
$ curl mongo-node.cloudfoundry.com
	Hello World
```

### Add MongoDB Configuration

In the previous step, you deployed the application and bound it to a Cloud Foundry mongodb service, although the application does not yet use the service.

In this step, you update the application to set up MongoDB connection information and credentials, whether the application is running locally or on the cloud.

* Add the following code to the beginning of `app.js`:

``` javascript

if(process.env.VCAP_SERVICES){
  var env = JSON.parse(process.env.VCAP_SERVICES);
  var mongo = env['mongodb-2.0'][0]['credentials'];
}
else{
  var mongo = {
    "hostname":"localhost",
    "port":27017,
    "username":"",
    "password":"",
    "name":"",
    "db":"db"
  }
}

var generate_mongo_url = function(obj){
  obj.hostname = (obj.hostname || 'localhost');
  obj.port = (obj.port || 27017);
  obj.db = (obj.db || 'test');

  if(obj.username && obj.password){
    return "mongodb://" + obj.username + ":" + obj.password + "@" + obj.hostname + ":" + obj.port + "/" + obj.db;
  }
  else{
    return "mongodb://" + obj.hostname + ":" + obj.port + "/" + obj.db;
  }
}

var mongourl = generate_mongo_url(mongo);
```

The `if` provides two different sets of information, depending on whether the application is running on the cloud or locally. `generate_mongo_url` creates appropriate connection information for MongoDB, which is then assigned to mongourl.

* Test the application locally:

``` bash
$ node app.js
```

* In another terminal:

``` bash
$ curl localhost:3000
```

The application returns the string "Hello World".

* Update the deployed cloud application:

``` bash
$ vmc update mongo-node
    Uploading Application:
      Checking for available resources: OK
      Packing application: OK
      Uploading (1K): OK
    Push Status: OK
    Stopping Application: OK
    Staging Application: OK
    Starting Application: OK
```

* Test the deployed application:

```bash
$ curl mongo-node.cloudfoundry.com
    Hello World
```

### Add MongoDB Functionality

Next, install the MongoDB native drivers locally and update the application to use MongoDB.

* Install MongoDB native drivers locally:

``` bash
$ npm install mongodb
```

This creates a new local directory, `node_modules`, in the application root.

* In `app.js`, create a new function, record_visit, that stores the server request to MongoDB:

``` javascript
var record_visit = function(req, res){
  /* Connect to the DB and auth */
  require('mongodb').connect(mongourl, function(err, conn){
    conn.collection('ips', function(err, coll){
      /* Simple object to insert: ip address and date */
      object_to_insert = { 'ip': req.connection.remoteAddress, 'ts': new Date() };

      /* Insert the object then print in response */
      /* Note the _id has been created */
      coll.insert( object_to_insert, {safe:true}, function(err){
        res.writeHead(200, {'Content-Type': 'text/plain'});
        res.write(JSON.stringify(object_to_insert));
        res.end('\n');
      });
    });
  });
}
```

The `.connect` method connects to MongoDB using either the local or Cloud Foundry mongourl.
Then the `.collection('ips', ...)` method adds the request information to the data that will be committed.

* Update the `http.createServer` method so that it calls the `record_visit` function when the server request is made:

``` javascript
    http.createServer(function (req, res) {
        record_visit(req, res);
    }).listen(port, host);
```

* Test the application locally:

``` bash
$ node app.js
```

and from another terminal:

``` bash
$ curl localhost:3000
    {"ip":"127.0.0.1","ts":"2011-12-29T23:22:38.192Z","_id":"4efcf63ecab9a5b41e000001"}
```

Press Control-C in the first terminal to stop the web server.

* Test the application on Cloud Foundry:

``` bash
$ vmc update mongo-node
$ curl appname.cloudfoundry.com
    {"ip":"127.0.0.1","ts":"2011-12-29T23:24:25.199Z","_id":"4efcf6a927996b5f79000001"}
```

* Create a function `print_visits` that prints the last ten visits/requests:

``` javascript
var print_visits = function(req, res){
/* Connect to the DB and auth */
require('mongodb').connect(mongourl, function(err, conn){
    conn.collection('ips', function(err, coll){
        coll.find({}, {limit:10, sort:[['_id','desc']]}, function(err, cursor){
            cursor.toArray(function(err, items){
                res.writeHead(200, {'Content-Type': 'text/plain'});
                for(i=0; i<items.length;i++){
                    res.write(JSON.stringify(items[i]) + "\n");
                }
                res.end();
            });
        });
    });
});
}
```

* Update the `createServer` method to call the new `print_visits` function:

``` javascript

http.createServer(function (req, res) {
    params = require('url').parse(req.url);
    if(params.pathname === '/history') {
        print_visits(req, res);
    }
    else{
        record_visit(req, res);
    }
}).listen(port, host);
```

Web server requests will either add the current visit to MongoDB (the default) or,
if url includes "/history", output the last ten visits.

* Test the application locally:

``` bash
$ curl localhost:3000
    {"ip":"127.0.0.1","ts":"2011-12-29T23:44:30.254Z","_id":"4efcfb5e2f9d30481f000003"}
$ curl localhost:3000/history
    {"ip":"127.0.0.1","ts":"2011-12-29T23:44:30.254Z","_id":"4efcfb5e2f9d30481f000003"}
    {"ip":"127.0.0.1","ts":"2011-12-29T23:31:39.339Z","_id":"4efcf85b2f9d30481f000002"}
    {"ip":"127.0.0.1","ts":"2011-12-29T23:31:26.678Z","_id":"4efcf84e2f9d30481f000001"}
    {"ip":"127.0.0.1","ts":"2011-12-29T23:22:38.192Z","_id":"4efcf63ecab9a5b41e000001"}
```

* Stop the application locally and update it on Cloud Foundry.

``` bash
$ vmc update mongo-node
    Uploading Application:
      Checking for available resources: OK
      Processing resources: OK
      Packing application: OK
      Uploading (8K): OK
    Push Status: OK
    Stopping Application: OK
    Staging Application: OK
    Starting Application: OK
```

* Test the cloud version of the application.

``` bash
$ curl mongo-node.cloudfoundry.com/history
    {"ip":"127.0.0.1","ts":"2011-12-29T23:49:46.738Z","_id":"4efcfc9acbfffadc0b000001"}
    {"ip":"127.0.0.1","ts":"2011-12-29T23:24:25.199Z","_id":"4efcf6a927996b5f79000001"}
```