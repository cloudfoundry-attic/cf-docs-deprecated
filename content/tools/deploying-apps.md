---
title: Deploying and Managing Applications
description: Managing Applications on Cloud Foundry
tags:
    - overview
    - vmc
    - sts
    - maven
    - roo
---

In the context of Cloud Foundry, deploying an application simply means making it available for users to start using.   When you deploy an application, you push the actual application bits (such as a WAR file in the case of Spring Java) to your target cloud, specify the required application configuration such as the deployment URL and required memory, and binding any required services such as MySQL.  Cloud Foundry then stages the application, starts it, and makes it available at the specified URL so users can use it.

**Subtopics**

+ [Using VMC](#using-vmc)
+ [Using STS](#using-sts)
+ [Using Maven](#using-maven)
+ [Using Roo](#using-roo)

## Using VMC

The following user scenarios are described in this documentation:

+ [Deploying an Application That Does Not Require a Service](#deploying-an-application-that-does-not-require-a-service)
+ [Deploying an Application That Requires One Service](#deploying-an-application-that-requires-one-service)
+ [Deploying an Application That Requires Multiple Services](#deploying-an-application-that-requires-multiple-services)
+ [Deploying in Non-Interactive Mode](#deploying-in-non-interactive-mode)
+ [Managing the Application Lifecycle (Stop, Start, Restart, Delete, Rename)](#managing-the-application-lifecycle-stop-start-restart-delete-rename)
+ [Updating The Deployment Bits](#updating-the-deployment-bits)
+ [Avoiding Downtime When Updating The Deployment Bits](#avoiding-downtime-when-updating-the-deployment-bits)
+ [Monitoring and Configuring A Deployment (Scaling and Memory Management)](#monitoring-and-configuring-a-deployment-scaling-and-memory-management)

## Deploying an Application That Does Not Require a Service

You use the `vmc push` command to deploy applications using VMC.  By default, `vmc` interactively prompts for the required deployment information; alternatively you can specify parameters to override the defaults or run in non-interactive mode.

The simplest deployment example is to run `vmc push` with no parameters to deploy an application that does not require any services to be bound to it.  The `vmc push` command will prompt for all required information.  The following example shows the output from deploying a very simple Spring application that is packaged in a WAR file called `hello.war`; see the text after the example for additional information:

```bash

    prompt$ vmc push

    Would you like to deploy from the current directory? [Yn]:
    Application Name: hello
    Application Deployed URL: 'hello.cloudfoundry.com'? hello-js.cloudfoundry.com
    Detected a Java SpringSource Spring Application, is this correct? [Yn]:
    Memory Reservation [Default:512M] (64M, 128M, 256M, 512M or 1G)
    Creating Application: OK
    Would you like to bind any services to 'hello'? [yN]:
    Uploading Application:
      Checking for available resources: OK
      Processing resources: OK
      Packing application: OK
      Uploading (4K): OK
    Push Status: OK

    Staging Application: OK
    Starting Application: OK

```

Go to the specified deployment URL in your browser to start using the application.  In the example above, that would be:

        http://hello-js.cloudfoundry.com

Note the following about the preceding `vmc push` example:

+ The default value of yes/no prompts is shown in capital letters at the end of the prompt; for example, if the default answer is No, then the prompt ends in `[yN]`.
+ You can deploy from the current directory in which you are running VMC, or specify a different directory. This directory is the location of the actual application bits, such as the `hello.war` file.   **Note** that `vmc push` deploys ALL artifacts in the specified directory, so be sure that only those files you want deployed are physically located in the directory.
+  The name of the application is the one you use to identify it as you mange its lifecycle, such as stopping it, adding memory to it, and so on.   The scope of the name is your Cloud Foundry account, which means the name should be unique as far as your account but does not need to be unique across the entire Cloud Foundry target.
+  By default, the deployment URL is the application name along with the target (for example, `cloudfoundry.com`).  However, you must be sure to specify a deployment URL that is *unique* across the entire Cloud Foundry target, even though the name of the application must be unique only within the scope of your Cloud Foundry account.  This is why, in the example above, the deployment URL includes an identifier that makes it unique across the entire cloudfoundry.com Web site.

## Deploying an Application That Requires One Service

The following example shows how to deploy an application that requires a service (in this case a MySQL service).   The application is called `hotels.war`.

In our example, we will first create an instance of the MySQL service using the `vmc create-service` command, and then bind this service instance to the application when we deploy it with `vmc push`.u

It is important to note that you do not need to change your existing application in any way for it to "find" the MySQL instance that Cloud Foundry created and to which the application is bound.  This is because Cloud Foundry uses an auto-reconfiguration mechanism that lets typical Spring applications consume services without changing anything in the application.

* Find the name of the MySQL service by running `vmc services` and noting the first column:

```bash

        prompt$ vmc services

        ============== System Services ==============

        +------------+---------+---------------------------------------+
        | Service    | Version | Description                           |
        +------------+---------+---------------------------------------+
        | mongodb    | 2.0     | MongoDB NoSQL store                   |
        | mysql      | 5.1     | MySQL database service                |
        | postgresql | 9.0     | vFabric Postgres database service     |
        | rabbitmq   | 2.4     | RabbitMQ messaging service            |
        | redis      | 2.2     | Redis key-value store service         |
        +------------+---------+---------------------------------------+

        =========== Provisioned Services ============

```

*  Create a new instance of the `mysql` service and call it `mysql-js`:

```bash
prompt$ vmc create-service mysql mysql-js
```

*  Deploy the `hotels.war` application, but this time bind it to the `mysql-js` service instance:

```bash

        prompt$ vmc push

        Would you like to deploy from the current directory? [Yn]: y
        Application Name: hotels
        Application Deployed URL: 'hotels.cloudfoundry.com'? hotels-js.cloudfoundry.com
        Detected a Java Web Application, is this correct? [Yn]:
        Memory Reservation [Default:512M] (64M, 128M, 256M, 512M or 1G)
        Creating Application: OK
        Would you like to bind any services to 'hotels'? [yN]: y
        Would you like to use an existing provisioned service [yN]? y
        The following provisioned services are available:
        1. mysql-js
        Please select one you wish to provision: 1
        Binding Service: OK
        Uploading Application:
          Checking for available resources: OK
          Packing application: OK
          Uploading (12K): OK
        Push Status: OK

        Staging Application: OK

        Starting Application: OK

```

You can also create a new instance of a service during the `vmc push` command by answer `N` to the prompt about using a provisioned service; in this case, the `vmc push` command prompts for information about the new service instance.

## Deploying an Application That Requires Multiple Services

The [preceding section](#deploying-an-application-that-requires-one-service) described how to deploy an application using the `vmc push` command and bind a service (MySQL) to it at the same time.  However, you were only able to bind *one* service, but what if your application requires multiple services?  This section describes how to perform that task.

This example extends the [previous example](#deploying-an-application-that-requires-one-service) in that, in addition to binding the `mysql-js` service to the `hotels` application, it will also create and bind the MongoDB and RabbitMQ services.

*  Create new instances of the `mysql`, `mongodb`, and `rabbitmq` services:

```bash

        prompt$ vmc create-service mysql mysql-js
        prompt$ vmc create-service mongodb mongodb-js
        prompt$ vmc create-service rabbitmq rabbitmq-js

```

*  Deploy the `hotels.war` application and bind the `mysql-js` service instance to it.  This time, however, be sure to specify the `--no-start` option so that Cloud Foundry does not stage or start the application:

```bash

        prompt$ vmc push --no-start

        Would you like to deploy from the current directory? [Yn]: y
        Application Name: hotels
        ...
        Uploading Application:
        ...
        Push Status: OK

```

* Use the `vmc bind-service` to bind the two remaining services to the `hotels` application:

```bash

        prompt$ vmc bind-service mongodb-js hotels
        prompt$ vmc bind-service rabbitmq-js hotels

```

*  Start the application:

```bash

        prompt$ vmc start hotels

        Staging Application: OK
        Starting Application: OK

```

## Deploying in Non-Interactive Mode

Use the `-n` option of `vmc push` to deploy an application in non-interactive mode.   The only other requirement is that you specify the name of the application.

If you specify no other options, then Cloud Foundry uses all the default values when deploying the application.  For example, if you execute the following command:

```bash

    prompt$ vmc push hotels -n

```

then Cloud Foundry does the following:

+ Deploys all artifacts in the current directory as part of the `hotels` application.
+ Uses the deployment URL`hotels.cloudfoundry.com` (assuming your target is the publicly-hosted Cloud Foundry.)
+ Creates a single instance of the deployed application.
+ Does *not* bind any services to the application.
+ Auto-detects the programming framework (such as Spring or Ruby)
+ Reserves 512 MB of memory for the application
+ Automatically starts the application after staging it.

You can use additional options to change the default values of the deployment, and still deploy in non-interactive mode.  For example, you can change the deployment URL using the `--url` option, the reserved memory using the `--mem` option, and specify that the application not start with the following command:

```bash

    prompt$ vmc push hotels -n --url hotels-js.cloudfoundry.com --mem 256 --no-start

```

See [VMC Quick Reference Guide](/tools/vmc/vmc-quick-ref.html#deploying-applications) for the full list of parameters you can specify for the `vmc push` command.

## Managing the Application Lifecycle (Stop, Start, Restart, Delete, Rename)

Managing the lifecycle of your application after you have deployed it is easy; just use the appropriate VMC command to stop, start, and restart it.  For example, to stop and then start an application:

```bash

    prompt$ vmc stop hotels
    prompt$ vmc start hotels

```

Or to do the same task with one command:

```bash

    prompt$ vmc restart hotels

```

It is important to note that when Cloud Foundry restarts an application, it creates an entire new virtual infrastructure for it.   Although this is typically transparent to the application, one situation in which this is relevant is if the application created local files on disk when it was previously running.  At restart, these files will be completely lost because Cloud Foundry creates a new virtual disk.  If the application requires that these files be available, even after a restart, then your application should use a data service such as MongoDB which will ensure that the files are always available.

To rename an application, use the `vmc rename` command, specifying the existing name and then the new name.  For example:

```bash
prompt$ vmc rename hotels hotels-new-name
```

The `vmc rename` command changes the internal name of the application; it does not change the deployment URL that you use to run the application.  For that you would use `vmc map`. See [Changing the Deployment URL](#changing-the-deployment-url).

To delete an application, and free up all resources associated with the application, use the `vmc delete` command:

```bash
prompt$ vmc delete hotels
```

## Updating The Deployment Bits

Use the `vmc update <appname>` command to update the "bits" that make up an application, such as the packaged WAR file for a Java Spring applications.

When Cloud Foundry updates an application, it first stops it, makes the change, and then starts it again.  This means that the application will be unavailable for a short period of time and existing user sessions will be deleted.  During the iterative development cycle of an application, this might not be a problem, but in production scenarios this may be unaccaptable.  See [Avoiding Downtime When Updating The Deployment Bits](#avoiding-downtime-when-updating-the-deployment-bits) for a useful procedure to avoid this short downtime.

Follow these steps to update your application:

* Use `vmc apps` to get the exact name of your application; the first column of the output lists the name.  For example:

```bash

        prompt$ vmc apps

        +-------------+----+---------+----------------------------+----------+
        | Application | #  | Health  | URLS                       | Services |
        +-------------+----+---------+----------------------------+----------+
        | hello       | 1  | RUNNING | hello-js.cloudfoundry.com  |          |
        | hotels      | 1  | RUNNING | hotels-js.cloudfoundry.com |          |
        +-------------+----+---------+----------------------------+----------+

```

* In your command prompt (Windows) or terminal window (Linux), change to the directory that contains your updated application bits.  Be sure the directory contains *only* the files and artifacts that you want to deploy because VMC pushes all files in the directory.

```bash
prompt$ cd /usr/bob/sample-apps/hotels
```

*  Use `vmc update` to update your application:

```bash
prompt$ vmc update hotels
```

    Check the output of the command to ensure the application started without errors, then start using the updated application.

## Avoiding Downtime When Updating The Deployment Bits

The method for updating an application without any downtime exploits the fact that in Cloud Foundry you can map a single application to multiple deployment URLs, and similarly, map multiple applications to the same deployment URL.   So rather than updating the application directly, we will create a new application, associate it with the existing URL, then *disassociate the mapping of the old application to the URL*, then delete the old application.

In the following example, the existing application is called `hotels` and its deployment URL is `hotels.cloudfoundry.com`, and you want to new user sessions to start using the new application seamlessly, without existing users losing their connection.

The exact steps are as follows:

* In your command prompt or terminal window, change to the directory that contains the updated application "bits".  For example:

```bash
prompt$ cd /usr/bob/sample-apps/hotels
```

*  Create a new application using the `vmc push` command, call it something different from the existing application, such as `hotels-new` and also specify that its deployment URL be something different, such as `hotels-new.cloudfoundry.com`:

```bash
prompt$ vmc push hotels-new
```

    Be sure you bind any required services, such as MySQL or RabbitMQ.  Use the same service instances that the existing application is currently using.

*  In a browser, go to the new deployment URL to test that the newly deployed application is working correctly, and that it is indeed the updated version.

    In our example, the deployment URL of the new application is `http://hotels-new.cloudfoundry.com`.

*  Associate the new application with the *existing* deployment URL using the `vmc map` command.  For example:

```bash
prompt$ vmc map hotels-new hotels.cloudfoundry.com
```

    At this point, because the same deployment URL maps to two different applications, new user sessions might go to *either* the old or the new application.

    You can test this by constantly refreshing the `http://hotels.cloudfoundry.com` URL; some of the time you will be directed to the old application, but some of the time to the new.  Be sure, however, that you *sometimes* are directed to the new version.

    You can also see this multiple URL association by executing `vmc apps`:

```bash

    prompt$ vmc apps

    +-------------+----+---------+-----------------------------------------+--------------+
    | Application | #  | Health  | URLS                                    | Services     |
    +-------------+----+---------+-----------------------------------------+--------------+
    | hotels      | 1  | RUNNING | hotels.cloudfoundry.com                 | mysql-hotels |
    | hotels-new  | 1  | RUNNING | hotels-new.cloudfoundry.com,            |              |
    |             |    |         |    hotels.cloudfoundry.com              | mysql-hotels |
    +-------------+----+---------+-----------------------------------------+--------------+

```

*  Disassociate the old application from its original deployment URL using the `vmc unmap` command:

```bash
prompt$ vmc unmap hotels hotels.cloudfoundry.com
```

    Note that this unmapping does *not* drop the existing user sessions to the old application; it simply stops new traffic to the old application so that new user sessions start using the new application.

* Test that the original deployment URL (`http://hotels.cloudfoundry.com` in our example) is now always directing user sessions to the new application.

*  Disassociate the new application from its new deployment URL:

```bash
prompt$ vmc unmap hotels-new hotels-new.cloudfoundry.com
```

*  When you are sure that there are no longer any user sessions connected to the old application, you can delete it:

```bash
prompt$ vmc delete hotels
```

    Be sure you do **not** delete any shared service instances that are currently being used by the new application.

    You will now have a new internal name for the application, but it maps to the existing deployment URL that users are already familiar with.  You can view this new mapping with `vmc apps`:

```bash

        prompt$ vmc apps

        +-------------+----+---------+-------------------------+-------------+
        | Application | #  | Health  | URLS                    | Services    |
        +-------------+----+---------+-------------------------+-------------+
        | hotels-new  | 1  | RUNNING | hotels.cloudfoundry.com | mysql-hotels|
        +-------------+----+---------+-------------------------+-------------+

```

## Monitoring and Configuring A Deployment (Scaling and Memory Management)

There are a number of VMC commands for configuring a deployed application, such as scaling it up or down by adding or removing instances, adding more memory, and changing its deployment URL.

Use the `vmc apps` command to get a list of all your deployed applications; you will use the name of the application to get more information about it and possibly configure it:

```bash

    prompt$ vmc apps

    +-------------+----+---------+----------------------------+----------+
    | Application | #  | Health  | URLS                       | Services |
    +-------------+----+---------+----------------------------+----------+
    | hello       | 1  | RUNNING | hello-js.cloudfoundry.com  |          |
    | hotels      | 1  | RUNNING | hotels-js.cloudfoundry.com | mysql-js |
    +-------------+----+---------+----------------------------+----------+

```

## Scaling The Application

Use the `vmc stats <appname>` command to display the current configuration and resource utilization of an application.  For example:

```bash

    prompt$ vmc stats hotels

    +----------+-------------+----------------+--------------+---------------+
    | Instance | CPU (Cores) | Memory (limit) | Disk (limit) | Uptime        |
    +----------+-------------+----------------+--------------+---------------+
    | 0        | 0.0% (4)    | 61.7M (512M)   | 8.4M (2G)    | 0d:0h:39m:42s |
    | 1        | 7.5% (4)    | 63.8M (512M)   | 8.4M (2G)    | 0d:0h:0m:22s  |
    +----------+-------------+----------------+--------------+---------------+

```

The table displays a row of data for each instance of the application; in the table above, there are two instances of `hotels`.   Each row shows the CPU utilization of the instance, the memory usage and the maximum allowed, the disk usage and the maximum, and how long the instance has been running.

To change the maximum allowed memory, use the `vmc mem` command, passing it the name of the application and the maximum memory in MB.  This changes the maximum for all instances of the application.  For example, to lower the maximum memory of the `hotels` application to 128 MB:

```bash
prompt$ vmc mem hotels 128
```

Note that Cloud Foundry must restart the application for this change to take effect.

Another way to scale the application up so that more user sessions can successfully and quickly connect at once, you can add more instances.  Conversely, if you want to conserve the resources associated with your Cloud Foundry account and scale *down* an application that currently has multiple instances running, you can delete instances.  For example, to specify that you want an application to have four instances running concurrently:

```bash
prompt$ vmc instances hotels 4
```

Use `vmc stats` to ensure that indeed there are four instances of the application running:

```bash

    prompt$ vmc stats hotels

    +----------+-------------+----------------+--------------+--------------+
    | Instance | CPU (Cores) | Memory (limit) | Disk (limit) | Uptime       |
    +----------+-------------+----------------+--------------+--------------+
    | 0        | 0.5% (4)    | 64.6M (128M)   | 8.4M (2G)    | 0d:0h:5m:4s  |
    | 1        | 0.5% (4)    | 62.4M (128M)   | 8.4M (2G)    | 0d:0h:5m:4s  |
    | 2        | 7.0% (4)    | 61.0M (128M)   | 8.4M (2G)    | 0d:0h:0m:22s |
    | 3        | 6.7% (4)    | 59.6M (128M)   | 8.4M (2G)    | 0d:0h:0m:22s |
    +----------+-------------+----------------+--------------+--------------+

```

Note that adding additional instances of a deployed application uses up more memory and disk space from your quota.

## Changing the Deployment URL

Use the `vmc map` and `vmc unmap` commands to change the deployment URL to which an application is associated.

For example, to associate a new  deployment URL to an existing application:

```bash
prompt$ vmc map hotels hotels-new.cloudfoundry.com
```

To unmap an existing URL, use the `vmc unmap` command:

```bash
prompt$ vmc unmap hotels hotels-new.cloudfoundry.com
```

See [Updating A Deployment Without Any Downtime](#updating-a-deployment-without-any-downtime) for additional examples of using these commands in a real-life scenario.

## Get Log and Crash Information about an Application

To see the most recent activity in the stderr.log and stdout.log files of the application, use the `vmc logs <appname>` command:

```bash
prompt$ vmc logs hotels
```

If you suspect that your application has crashed recently, run `vmc crashes <appname>` which will provide you with more information:

```bash
prompt$ vmc crashes hotels
```

Use `vmc crashlogs` to get more detailed information about the crash:

```bash
prompt$ vmc crashlogs hotels
```
