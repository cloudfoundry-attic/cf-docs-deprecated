---
title: Play Java
description: Play Java Application Development with Cloud Foundry
tags:
    - play
    - java
    - tutorial
---

This is a guide for Java developers using the Play framework to build their apps and
deploy on Cloud Foundry. It shows you how to set up and successfully deploy Play Java applications
to Cloud Foundry.

Play is based on a lightweight, stateless, web-friendly architecture and features predictable
and minimal resource consumption (CPU, memory, threads) for highly-scalable applications.
Cloud Foundry provides a runtime environment for Play 2.0 applications and the Cloud Foundry
deployment tools automatically recognize Play applications.

Before you get started, you need the following:

+  A [Cloud Foundry account](http://cloudfoundry.com/signup)

+  The [vmc](/tools/vmc/installing-vmc.html) Cloud Foundry command line tool

+  A [Play 2.0 ](http://www.playframework.org/documentation/2.0.2/Home) installation

## Deploying Play Java Applications to Cloud Foundry

When you deploy a Play application to Cloud Foundry, the current directory must contain
the application, the conf folder and app folders which are used by Cloud Foundry to detect
a Play application.

Here are steps to create and deploy a "hello world" Play application.

### Install Play

Please follow the instructions given on the [Play website](http://www.playframework.org/documentation/2.0.2/Installing)
to download and install Play 2.0.

Make sure the play script is in your path. On a Linux or a Mac environment this can be done using

``` bash
export PATH=$PATH:/path/to/play20
```
On Windows you’ll need to set it in the global environment variables.

### Check the play command is available
Run the `play help` command in the shell.

``` bash
$play help
```
The Following screen should show up.

![play-help.png](/images/screenshots/play/play-help.png)

### Create the App

To create the app, type the following command:

``` bash
$ play new helloworld-java

```
Choose the Appropriate template (Java in our case).

``` bash
$play new helloworld-java
       _            _
 _ __ | | __ _ _  _| |
| '_ \| |/ _' | || |_|
|  __/|_|\____|\__ (_)
|_|            |__/

play! 2.0.2, http://www.playframework.org

The new application will be created in /Users/[username]/play/mysamples/helloworld-java

What is the application name?
> helloworld-java

Which template do you want to use for this new application?

  1 - Create a simple Scala application
  2 - Create a simple Java application
  3 - Create an empty project

> 2

OK, application helloworld-java is created.

Have fun!
```

The following directory structure gets created:

```bash
$cd helloworld-java
$ls
README
app
conf
project
public
```
This will create a new directory `helloworld-java`.

### Run the Default App Locally

Try running the app locally with the following command inside the play console:

```bash
[helloworld-java] $ run

[info] Updating {file:/Users/[username]/play/mysamples/helloworld-java/}helloworld...
[info] Done updating.
--- (Running the application from SBT, auto-reloading is enabled) ---

[info] play - Listening for HTTP on port 9000...

(Server started, use Ctrl+D to stop and go back to the console...)
```
You should see the default Play template being served at `http://localhost:9000`.

![default-play-home.png](/images/screenshots/play/default-play-home.png)

### Modify the Controller

In this step, we will modify the controller `helloworld-java/app/controllers/Application.java`

```java
package controllers;

import play.*;
import play.mvc.*;

import views.html.*;

public class Application extends Controller {

  public static Result index() {
    return ok("HelloWorld Java");
  }
}

```
The default view in the browser would look like this:

![play-helloworld-java.png](/images/screenshots/play/play-helloworld-java.png)


### Deploy the App

Build the Play distribution using the following command:

```bash
$play dist
[info] Loading project definition from /Users/[username]/vmware/play/mysamples/helloworld-java/project
[info] Set current project to helloworld (in build file:/Users/[username]/vmware/play/mysamples/helloworld-java/)
[info] Packaging /Users/[username]/vmware/play/mysamples/helloworld-java/target/scala-2.9.1/helloworld_2.9.1-1.0-SNAPSHOT.jar ...
[info] Done packaging.

Your application is ready in /Users/[username]/vmware/play/mysamples/helloworld-java/dist/helloworld-1.0-SNAPSHOT.zip

[success] Total time: 3 s, completed Jul 26, 2012 10:55:56 PM
```
Target Cloud Foundry and log in with your Cloud Foundry credentials:

```bash
$ vmc target api.cloudfoundry.com
$ vmc login

```

Push the application. You can press `Enter` to accept the defaults at most of the prompts,
but be sure to enter a unique URL for the application. Here is an example push:

``` bash
$vmc push --path=dist/helloworld-1.0-SNAPSHOT.zip
Application Name: helloworld-java
Detected a Play Framework Application, is this correct? [Yn]: Y
Application Deployed URL [helloworld-java.cloudfoundry.com]:
Memory reservation (128M, 256M, 512M, 1G, 2G) [256M]:
How many instances? [1]: 1
Bind existing services to 'helloworld-java'? [yN]: N
Create services to bind to 'helloworld-java'? [yN]: N
Would you like to save this configuration? [yN]: y
Manifest written to manifest.yml.
Creating Application: OK
Uploading Application:
  Checking for available resources: OK
  Processing resources: OK
  Packing application: OK
  Uploading (66K): OK
Push Status: OK
Staging Application 'helloworld-java': OK
Starting Application 'helloworld-java': OK
```

Notice how vmc was able to detect a Play Application automatically.

Access the application with your browser at the specified URL,
[http://helloworld-java.cloudfoundry.com](http://helloworld-java.cloudfoundry.com) in this example.

![play-helloworld-java-cf.png](/images/screenshots/play/play-helloworld-java-cf.png)


## Next Steps

+    [Java App with MySQL](/frameworks/play/java-mysql.html)
+    [Java App with PostgreSQL](/frameworks/play/java-postgresql.html)