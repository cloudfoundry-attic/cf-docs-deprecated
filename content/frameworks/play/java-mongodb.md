---
title: Play Java MongoDB App on Cloud Foundry
description: Play Java Application with MongoDB Backend Deployed on Cloud Foundry
tags:
    - play
    - java
    - mongodb
    - tutorial
---

This is a guide for Java developers using the Play framework and MongoDB NoSQL database to
build and deploy their apps on Cloud Foundry. It shows you how to set up and acces MongoDB using
native Java driver. The App uses a local instance of MongoDB driver for development and an
existing MongoDB service running on Cloud Foundry


Before you get started, you need the following:

+  A [Cloud Foundry account](http://cloudfoundry.com/signup)

+  The [vmc](/tools/vmc/installing-vmc.html) Cloud Foundry command line tool

+  A [Play 2.0 ](http://www.playframework.org/documentation/2.0.2/Home) installation

+  A Local [MongoDB](http://www.mongodb.org/downloads)  Server and Client setup

## Introduction

In this tutorial, we take a simple use case:

Modifying and Deploying the [TodoList]( http://www.playframework.org/documentation/2.0.2/JavaTodoList )
app to Cloud Foundry with the persistent layer wired to a MongoDB NoSQL database. Since the original
app was written for a SQL datastore using Ebeans, we will rewrite the controller and the model
of this app to work with the MongoDB back end.

## TodoList Play App

TodoList is used to create and manage tasks.

![todolist-usecase.png](/images/play/todolist-usecase.png)

These tasks are created by the users and the data
is stored in underlying relational store.

High Level Interaction diagram between the Application using Play runtime and MongoDB can be
found below:

![play-mongodb.png](/images/play/play-mongodb.png)

### Steps to Create the App
In this section we outline the steps to create the App from the default Play app template.

#### Create a New Play App

Type the command `play new todolist-java-mongodb` on the command line. Choose Option 2 to create
a simple Java application.

```bash
$ play new todolist-java-mongodb
       _            _
 _ __ | | __ _ _  _| |
| '_ \| |/ _' | || |_|
|  __/|_|\____|\__ (_)
|_|            |__/

play! 2.0.2, http://www.playframework.org

The new application will be created in /Users/rajdeepd/vmware/play/mysamples/todolist-java-mongodb

What is the application name?
> todolist-java-mongodb

Which template do you want to use for this new application?

  1 - Create a simple Scala application
  2 - Create a simple Java application
  3 - Create an empty project

> 2

OK, application todolist-java-mongodb is created.

Have fun!
```

#### Create an Eclipse Project

Use the eclipsify command of Play to create an Eclipse project.

```bash
$ cd todolist-java-mongodb
$ play
[info] Loading project definition from /Users/<username>/play/mysamples/
todolist-java-mongodb/project
[info] Set current project to todolist-java-mongodb (in build file:/Users/<username>/play/mysamples/todolist-java-mongodb/)
       _            _
 _ __ | | __ _ _  _| |
| '_ \| |/ _' | || |_|
|  __/|_|\____|\__ (_)
|_|            |__/

play! 2.0.2, http://www.playframework.org

> Type "help play" or "license" for more information.
> Type "exit" or use Ctrl+D to leave this console.

[todolist-java-mongodb] $ eclipsify
[info] About to create Eclipse project files for your project(s).
[info] Updating {file:/Users/<username>/play/mysamples/todolist-java-mongodb/}todolist-java-mongodb...
[info] Done updating.
[info] Compiling 4 Scala sources and 2 Java sources to /Users/<username>/play/mysamples/
todolist-java-mongodb/target/scala-2.9.1/classes...
[info] Successfully created Eclipse project files for project(s):
[info] todolist-java-mongodb
[todolist-java-mongodb] $
```
Import the created Project into Eclipse using `File ->Import` and point to the root folder of
the project `todolist-java-mongodb`.

![eclipse-import-mongoproject-1.png](/images/screenshots/play/eclipse-import-mongoproject-1.png)

![eclipse-import-mongoproject-2.png](/images/screenshots/play/eclipse-import-mongoproject-2.png)

#### Add the MongoDB Driver and Gson Jar to the Project

Download and add the MongoDB driver file and Gson Jar to the lib folder of the project.

  + [mongo-0.2.8.jar](http://www.mongodb.org/downloads)
  + [gson-2.2.2.jar](http://code.google.com/p/google-gson/downloads/list)

#### Modify the Router
Define the following routes in the `routes` configuration file. It maps the relative URLs
 to the appropriate actions in `controllers.Application.java`

##### Index Page
Mapping below maps `HTTP GET` call to URL `/` to index.html page

``` javascript
GET     /                           controllers.Application.index()
```

##### Get All Tasks
Mapping below lists all the tasks using `HTTP GET` call to URL `/tasks`

``` javascript
GET     /tasks                  controllers.Application.tasks()
```

##### Create a Task
Mapping below maps `controllers.Application.newTask()` controller action to `HTTP POST` call to URL `/tasks`.

``` javascript
POST    /tasks                  controllers.Application.newTask()
```

##### Delete a Task
Mapping below maps `controllers.Application.deleteTask(:id Long)` controller action to
`HTTP POST` call to URL `/tasks/:id/delete` where id is the unique id for the task.

``` javascript
POST    /tasks/:id/delete        controllers.Application.deleteTask(id: Long)
```

##### Complete Listing


``` javascript
# Home page
GET     /                       controllers.Application.index()

# Map static resources from the /public folder to the /assets URL path
GET     /assets/*file           controllers.Assets.at(path="/public", file)

# Tasks
GET     /tasks                  controllers.Application.tasks()
POST    /tasks                  controllers.Application.newTask()
POST    /tasks/:id/delete       controllers.Application.deleteTask(id: Long)
```

#### Modify the Model
Create a class Task in models package which will interface with the underlying MongoDB driver.
Task has two public fields `id` and 'label`.
``` java
package models;

public class Task {
  public Long id;
  public String label;

}
```

#### Modify the Controller
In this section you will execute following tasks to implement the Controller functionality.

##### Create Action Methods
Open the controller file and add the following actions.

``` java
public class Application extends Controller {

  public static Result index() {
      return tasks();
  }

  public static Result tasks() {

  }

  public static List<Task> getTaskList() {

  }

  public static Result newTask() {
     
  }
}
```

##### Add Utility Method to access MongoDB
This method returns the local or remote instance of a MongoDB depending on the boolean
parameter `local`.

For local instance set `local=true` and `local=false` for remote MongoDB when we deploy it
to cloudfoundry.com
``` java
private static DB getDb() {
    String userName = play.Configuration.root().getString("mongo.remote.username");
    String password = play.Configuration.root().getString("mongo.remote.password");
    String langs = play.Configuration.root().getString("application.langs");
    boolean local  = true;

    String localHostName = play.Configuration.root().getString("mongo.local.hostname");
    Integer  localPort = play.Configuration.root().getInt("mongo.local.port");

    String remoteHostName = play.Configuration.root().getString("mongo.remote.hostname");
    Integer remotePort = play.Configuration.root().getInt("mongo.remote.port");

    Mongo m;
    DB db = null;
    if(local){
        String hostname = localHostName;
        int port = localPort;
        try {
            m = new Mongo( hostname, port);
            db = m.getDB( "db" );
        }catch(Exception e) {
            Logger.error("Exception while intiating Local MongoDB", e);
        }
    }else {
        String hostname = remoteHostName;
        int port = remotePort;
        try {
            m = new Mongo( hostname , port);
            db = m.getDB( "db" );
            boolean auth = db.authenticate(userName, password.toCharArray());
        }catch(Exception e) {
            Logger.error("Exception while intiating Local MongoDB", e);
        }
    }
    return db;
}
```

#### Add Parameters to application.config
The following parameters are used by the MongoDB driver to connect to a MongDB server either
on a local machine or inside cloudfoundry.com

``` javascript
mongo.local.hostname=localhost
mongo.local.port=27017

mongo.remote.hostname="172.30.XX.XX"
mongo.remote.port=25189
mongo.remote.username=<username obtained from caldecott>
mongo.remote.password=<password obtained from caldecott>
```

## Build and Run the App Locally
Before you run the client locally make sure the MongoDB is running locally.

``` bash
$ ./mongod --dbpath=../../mongo/db/
Thu Aug 30 18:24:09 [initandlisten] MongoDB starting : pid=45601 port=27017 dbpath=
../../mongo/db/ 64-bit host=username.local
Thu Aug 30 18:24:09 [initandlisten] db version v2.0.6, pdfile version 4.5
Thu Aug 30 18:24:09 [initandlisten] git version: e1c0cbc25863f6356aa4e31375add7bb49fb05bc
Thu Aug 30 18:24:09 [initandlisten] build info: Darwin erh2.10gen.cc 9.8.0 Darwin Kernel s
Version 9.8.0: Wed Jul 15 16:55:01 PDT 2009; root:xnu-1228.15.4~1/
RELEASE_I386 i386 BOOST_LIB_VERSION=1_40
Thu Aug 30 18:24:09 [initandlisten] options: { dbpath: "../../mongo/db/" }
Thu Aug 30 18:24:09 [initandlisten] journal dir=../../mongo/db/journal
Thu Aug 30 18:24:09 [initandlisten] recover : no journal files present, no recovery needed
Thu Aug 30 18:24:09 [websvr] admin web console waiting for connections on port 28017
Thu Aug 30 18:24:09 [initandlisten] waiting for connections on port 27017

```
The app can be run locally using the standard command from within the app folder.

``` bash
$play run
```
The following screen shows at when the browser is pointed to `http://localhost:9000`.

![play-todolist-java-mongodb-local.png](/images/screenshots/play/play-todolist-java-mongodb-local.png)

You can connect to the local MongoDB instance using `mongo` command line shell and query the collection.
`mongo` is shell binary. Exact location might vary depending on installation method and platform.

``` bash
$ ./mongo
MongoDB shell version: 2.0.6
connecting to: test
> use db;
switched to db db
> db.tasklist.find();
{ "_id" : ObjectId("50403ee86970ef63a14dea87"), "id" : NumberLong(0), "label" : "Drink Coffee" }
{ "_id" : ObjectId("50403efc6970ef63a14dea88"), "id" : NumberLong(1), "label" : "Write Play tutorial" }
{ "_id" : ObjectId("504044386970ef63a14dea89"), "id" : NumberLong(2), "label" : "" }
>
```

## Deploy the App on Cloud Foundry
Deployment to Cloud Foundry can be done in two simple steps:

+  Create a distribution
+  Push the distribution using `vmc push`

Commands below show the exact commands and output when the app is deployed.

``` bash
$ play dist
[info] Loading project definition from /Users/<username>/vmware/play/mysamples/
todolist-java-mongodb/project
[info] Set current project to todolist-java-mongodb (in build file:/Users/<username>/
vmware/play/mysamples/todolist-java-mongodb/)
[info] Compiling 1 Java source to /Users/<username>/vmware/play/mysamples/
todolist-java-mongodb/target/scala-2.9.1/classes...
[info] Packaging /Users/<username>/vmware/play/mysamples/todolist-java-mongodb/target/
scala-2.9.1/todolist-java-mongodb_2.9.1-1.0-SNAPSHOT.jar ...
[info] Done packaging.

Your application is ready in /Users/<username>/vmware/play/mysamples/todolist-java-mongodb/dist/
todolist-java-mongodb-1.0-SNAPSHOT.zip

[success] Total time: 8 s, completed Aug 31, 2012 12:06:54 AM

$ cd dist

$ vmc push
Would you like to deploy from the current directory? [Yn]: Y
Application Name: todolist-java-mongodb
Detected a Play Framework Application, is this correct? [Yn]: Y
Application Deployed URL [todolist-java-mongodb.cloudfoundry.com]:
Memory reservation (128M, 256M, 512M, 1G, 2G) [256M]:
How many instances? [1]: 1
Bind existing services to 'todolist-java-mongodb'? [yN]: y
1: mongodb-a1c55
2: mysql-2c653
3: mysql-2ea2b
4: mysql-c92b7
5: mysql-eeb13
6: mysql-first_app_ruby
7: postgresql-9a20a
8: rabbitmq-b25e8
Which one?: 1
Bind another? [yN]: N
Create services to bind to 'todolist-java-mongodb'? [yN]: N
Would you like to save this configuration? [yN]: y
Manifest written to manifest.yml.
Creating Application: OK
Binding Service [mongodb-a1c55]: OK
Uploading Application:
  Checking for available resources: OK
  Processing resources: OK
  Packing application: OK
  Uploading (84K): OK
Push Status: OK
Staging Application 'todolist-java-mongodb': OK
Starting Application 'todolist-java-mongodb': OK


```

Upon completion of deployment, we can go and visit the actual app at the URL `[app-name].cloudfoundry.com]`

![play-todolist-java-mongodb.png](/images/screenshots/play/play-todolist-java-mongodb.png)

## Summary
In this tutorial, we learned how to build and deploy a basic Play Java app using MongoDB as the backend
using a native MongoDB driver.

