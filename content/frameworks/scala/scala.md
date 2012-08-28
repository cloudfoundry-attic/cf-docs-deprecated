---
title: Scala
description: Scala Application Development with Cloud Foundry
tags:
    - scala
    - code-attached
---

[Scala](http://www.scala-lang.org/) is a statically typed, JVM-based language that combines object-oriented and functional programming in a consistent manner. It allows writing compact and robust programs for greater focus on core business logic. As a JVM-based language it offers easy interoperability with the Java ecosystem and Java libraries (such as Spring). Further, Scala incorporates many innovations such as parallel programming and the actor-based concurrent programming model to allow developers to develop modern applications and easily take advantage of multi-core hardware.

Most Scala applications written to [Lift](http://liftweb.net/) and [Spring](http://springframework.org/) will deploy seamlessly without modification to Cloud Foundry.  Other Scala applications may require minor modifications to bind with application services in the cloud.

## Can I use Lift on Cloud Foundry?

[Yes](/frameworks/scala/lift.html)

## Scala Support in Action

Cloud Foundry simplifies application deployment so you can focus on application development. You can use the Spring Tool Suite (STS) or the vmc command-line tool to deploy Scala applications.

Here is an example of deploying the [PocketChange](https://github.com/cloudfoundry-samples/pocketchangeapp-cf-runtime) application to Cloud Foundry using vmc. For most prompts, you just press Enter to accept the default.

### Build the app and upload it to Cloud Foundry

Note that in the step below we can also bind a relational database service if the app uses one.

```bash

$ mvn package
...

$ vmc push pocketchange --path target

    Application Deployed URL: 'pocketchange.cloudfoundry.com'?
    Detected a Scala Lift Application, is this correct? [Yn]:
    Memory Reservation [Default:512M] (64M, 128M, 256M, 512M, 1G or 2G)
    Creating Application: OK

    Would you like to bind any services to 'pocketchange'? [yN]: y
    The following system services are available:
    1. mongodb
    2. mysql
    3. redis
    Please select one you wish to provision: 2
    Specify the name of the service [mysql-744ef]: pocketchange-db
    Creating Service: OK
    Binding Service: OK

    Uploading Application:
      Checking for available resources: OK
      Processing resources: OK
      Packing application: OK
      Uploading (4K): OK
    Push Status: OK
    Staging Application: OK
    Starting Application: OK

```

During this deployment process, application bits are transferred to the cloud. (Notice the small upload size? This is due to the incremental update feature of Cloud Foundry). Once transferred, the application bits are wrapped in a Tomcat container, services you provisioned are bound (pocketchange-db, in this case), and a property file is created with information needed to connect to the database.

### Ready
The application is now ready to view at [http://pocketchange.cloudfoundry.com](http://pocketchange.cloudfoundry.com).

## How Does it Work?

Since Scala is a JVM-based language, you can write applications with any Java web framework or use a framework explicitly designed for Scala such as Lift. Cloud Foundry can now deploy such applications. This support is multi-faceted.

If a Scala web app uses no application services, such as a database, you can deploy it without any changes. This is true regardless of the framework used, or even if no framework was used. For example, see [http://hello-lift.cloudfoundry.com](http://hello-lift.cloudfoundry.com), which is a basic Lift application.

Most applications do use services, and in these cases you will need to make a few changes.

Scala applications using the Lift framework and the Lift guidelines to externalize services will deploy without modification. ([See the modified PocketChange app for an example.](https://github.com/cloudfoundry-samples/pocketchangeapp-prop)) You simply create a WAR file and deploy it.

For Lift applications that do not externalize their services, it is possible to explicitly access them through code. [See another modification to the PocketChange app:](https://github.com/cloudfoundry-samples/pocketchangeapp-cf-runtime)

```scala

import org.cloudfoundry.runtime.env._
import org.cloudfoundry.runtime.service.relational._
  Full(new MysqlServiceCreator(
    new CloudEnvironment()).createSingletonService().service.getConnection())

```

The application deployed at [http://tpuf.cloudfoundry.com](http://tpuf.cloudfoundry.com) is a Lift application that explicitly declares its use of a Redis service. See its [github repository](https://github.com/dcbriccetti/talking-puffin/) for more details.

If a Scala application uses the Spring framework, it can leverage auto-reconfiguration support and require no changes to the application itself. The application deployed at [http://hello-spring-scala.cloudfoundry.com](http://hello-spring-scala.cloudfoundry.com/) is a basic Spring application written using Scala with Spring as the framework.
