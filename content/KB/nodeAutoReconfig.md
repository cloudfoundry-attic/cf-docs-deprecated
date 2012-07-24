## Auto-reconfiguration for Node.js applications
Auto-reconfiguration is a Cloud Foundry's internal feature that automatically
configures connection settings of an app while running on Cloud Foundry. These Connections settings could be App's port or URI to connect to a service (like MongoDB or Redis etc).

This enables users who are running their app on localhost:3000 & say connected to mongodb mongodb://localhost/myDB to push their app to Cloud Foundry without making any changes to their code.

Let's take a look at 'before' & 'after' auto-reconfiguration to see how it works.

### Port Example: BEFORE Auto-reconfiguration

Notice that earlier user had to get port from `process.env.VCAP_APP_PORT` environment variable and make changes to the code
in order to run it on Cloud Foundry.

```javascript

var http = require('http');
var port = process.env.VCAP_APP_PORT || '3000';
http.createServer(function(req, res){
	res.end('Hello World!');
}).listen(port);

```

### Port Example: AFTER Auto-reconfiguration
Now, with auto-reconfiguration, user can upload the code that was running on localhost with a hard-coded port (3000 in this case), without making any changes to the code-itself. And it will still work on Cloud Foundry!

```javascript

var http = require('http');
http.createServer(function(req, res){
	res.end('Hello World!');
}).listen(3000);

```




Now, let's take a look at how powerful auto-configuration is by using a MongoDB example.

### MongoDB Example: BEFORE Auto-reconfiguration
In the below example, you can see that we had a rather messy `generate_mongo_url` function that figures out what mongoDB's url should be when the app runs in Cloud Foundry and also in localhost. 

```javascript

var port = (process.env.VMC_APP_PORT || 3000);
var host = (process.env.VCAP_APP_HOST || 'localhost');
var http = require('http');

if(process.env.VCAP_SERVICES){
  var env = JSON.parse(process.env.VCAP_SERVICES);
  var mongo = env['mongodb-1.8'][0]['credentials'];
}
else {
  var mongo = {
    "hostname":"localhost",
    "port":27017,
    "username":"",
    "password":"",
    "name":"",
    "db":"test"
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
console.log(mongourl);

http.createServer(function (req, res) {
  params = require('url').parse(req.url);
  if(params.pathname === '/history') {
    print_visits(req, res);
  }
  else{
    record_visit(req, res);
  }
}).listen(port, host);

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

var print_visits = function(req, res){
  /* Connect to the DB and auth */
  require('mongodb').connect(mongourl, function(err, conn){
    conn.collection('ips', function(err, coll){
      /*find with limit:10 & sort */
      coll.find({}, {limit:10, sort:[['_id','desc']]}, function(err, cursor){
        cursor.toArray(function(err, items){
          res.writeHead(200, {'Content-Type': 'text/plain'});
          for(i=0; i < items.length; i++){
            res.write(JSON.stringify(items[i]) + "\n");
          }
          res.end();
        });
      });
    });
  });
}

```


### MongoDB Example: AFTER Auto-reconfiguration

With Auto-reconfig, users don't need to add code to figure out what the mongoDB's url should be. They can simply upload their code that's running on 
localhost to Cloud Foundry, and it will just work. 

PS: Notice `var mongoUrl = 'mongodb://localhost:27017/test';` is still pointing to mongodb on localhost and that `generate_mongo_url` function is gone.


```javascript

var http = require('http');

//With auto-reconfig, we don't have to change localhost's mongodb url
var mongoUrl = 'mongodb://localhost:27017/test';

http.createServer(function (req, res) {
  params = require('url').parse(req.url);
  if(params.pathname === '/history') {
    print_visits(req, res);
  }
  else{
    record_visit(req, res);
  }
}).listen(3000);

var record_visit = function(req, res){
  /* Connect to the DB and auth */
  require('mongodb').connect(mongoUrl, function(err, conn){
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

var print_visits = function(req, res){
  /* Connect to the DB and auth */
  require('mongodb').connect(mongourl, function(err, conn){
    conn.collection('ips', function(err, coll){
      /*find with limit:10 & sort */
      coll.find({}, {limit:10, sort:[['_id','desc']]}, function(err, cursor){
        cursor.toArray(function(err, items){
          res.writeHead(200, {'Content-Type': 'text/plain'});
          for(i=0; i < items.length; i++){
            res.write(JSON.stringify(items[i]) + "\n");
          }
          res.end();
        });
      });
    });
  });
}

```

### Under the hood

When your application is staged during the deployment process, Cloud Foundry will make two modifications:

* It will provide cf-autoconfig node module to your application.
* It will preload cf-autoconfig module while bootstrapping your application.
* Cf-autoconfig module exploits Node.js caching mechanism. This module searches application files for supported node modules and redefines module connection functions so that they are being called with Cloud Foundry connection parameters. For example, let’s see how it redefines connect function of mongodb node module:

```javascript

if ("connect" in moduleData) {
  var oldConnect = moduleData.connect;
  var oldConnectProto = moduleData.connect.prototype;
 
  moduleData.connect = function () {
    var args = Array.prototype.slice.call(arguments);
    args[0] = props.url;
    return oldConnect.apply(this, args);
  };
  moduleData.connect.prototype = oldConnectProto;
}
``` 

Other functions are redefined the same way. Please take a look at cf-autoconfig module source on github, feel free to provide a feedback or even a pull request.

### Supported modules

The following is the list of supported modules:

* [amqp](https://github.com/postwait/node-amqp)
* [mongodb](https://github.com/mongodb/node-mongodb-native)
* [mongoose](https://github.com/learnboost/mongoose)
* [mysql](https://github.com/felixge/node-mysql)
* [pg](https://github.com/brianc/node-postgres)
* [redis](https://github.com/mranney/node_redis)

According to [search.npmjs.org](search.npmjs.org) these modules are the most dependent on. This means there are many other modules that use these modules for database connection layer and auto-reconfiguration will be applied to them as well.

### Limitations

Auto-reconfiguration of services work only under the following conditions:

* You are using only one service of the given type. For example, one mysql or one redis service.
* You are using service node module from the list of supported modules above, or any other that is using one of them for service connection underneath.
* Your application does not use cf-runtime or cf-autoconfig node modules.
* Your application is a typical Node.js application. For a complex application you may need to consider to opt-out of auto-reconfiguration and use cf-runtime node module described in the next blog post in these series.
Opting out of auto-reconfiguration

### Turning-off Auto-reconfiguration
Auto-reconfiguration can be turned off by creating `cloudfoundry.json` file in application base folder with the option “cfAutoconfig” set as false.

```javascript

{ “cfAutoconfig” : false }
```
In addition, as mentioned above, auto-reconfiguration will not work if application is using cf-runtime node module.


### For More:
Please look at the blog here: [Auto-reconfiguration for Node.js applications](http://wp.me/p2wH9O-BS)








