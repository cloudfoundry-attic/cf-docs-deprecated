---
title: Node.js
description: Deploying Whiteboard App to Cloud Foundry
tags:
    - nodejs
    - express
    - tutorial
---

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
 Application Name: nodejs-whiteboard
 Application Deployed URL [nodejs-whiteboard.cloudfoundry.com]:
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

Access the application with your browser at the specified URL, [http://nodejs-whiteboard.cloudfoundry.com](http://nodejs-whiteboard.cloudfoundry.com) in this example.

## Changing the version of Node your App uses

You can use the runtime flag and pass the desired runtime. For how to list the runtimes see the next section

```bash
Example to use Node 0.6.8
```