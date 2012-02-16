---
title: VMC Quick Reference
description: All of the vmc commands available
tags:
    - vmc
    - CLI
---

This section groups the main VMC commands into functional categories and provides typical usage.   In the examples, text displayed like `<this>` indicates a variable whose value is specific to your environment.

Run `vmc help` to view the complete list of VMC commands, along with their parameters and a brief description.

## Subtopics

+ [Getting the Latest Version of VMC](#getting-the-latest-version-of-vmc)
+ [Suppressing Prompts to Run In Non-Interactive Mode](#suppressing-prompts-to-run-in-non-interactive-mode)
+ [Targeting Cloud Foundry](#targeting-cloud-foundry)
+ [Logging In and Managing Accounts](#logging-in-and-managing-accounts)
+ [Deploying Applications](#deploying-applications)
+ [Updating a Deployment](#updating-a-deployment)
+ [Managing the Deployment Lifecycle (Start, Stop, Delete)](#managing-the-deployment-lifecycle-start-stop-delete)
+ [Managing and Binding Services](#managing-and-binding-services)
+ [Getting Information about the Cloud Foundry Target](#getting-information-about-the-cloud-foundry-target)
+ [Getting Information about Applications](#getting-information-about-applications)

## Getting the Latest Version of VMC

`vmc` is provided as a Ruby gem, and is being constantly updated with new commands or new options to existing commands.  For this reason, be sure you have the latest version by executing the following RubyGems command:

```bash
prompt$ gem update vmc
```

##Suppressing Prompts to Run In Non-Interactive Mode

By default, `vmc` operates in interactive mode and the execution of many commands results in multiple prompts for values for specific options. To use `vmc` in non-interactive mode and provide options in the command itself, use the following syntax:

```bash
prompt$ vmc *command* -n --options
```

For example, to deploy an application and take the default value for all prompts:

```bash
prompt$ vmc push &lt;appname&gt; -n
```

To display all commands and options which can be used as parameters on the command line:

```bash
prompt$ vmc help options
```

## Targeting Cloud Foundry

Target Cloud Foundry in the Cloud:

```bash
prompt$ vmc target api.cloudfoundry.com
```

Target a standalone a Micro Cloud Foundry running on your local virtual machine:

```bash
prompt$ vmc target api.&lt;domain&gt;.cloudfoundry.com
```

**Note**: You specified the value of *domain* when you initially installed Micro Cloud Foundry.

Display the current target URL:

```bash
prompt$ vmc target
```

Display the list of known targets and the associated authorization tokens:

```bash
prompt$ vmc targets
```

## Logging In and Managing Accounts

Identify yourself to Cloud Foundry with your account information:

```bash
prompt$ vmc login &lt;youremail@email.com&gt; --passwd &lt;yourpassword&gt;
```

Change your password:

```bash
prompt$ vmc passwd
```

Logout from your current session:

```bash
prompt$ vmc logout
```

Get information about your Cloud Foundry account, the version of VMC you are using, and the total resources you have consumed:

```bash
prompt$ vmc info
```

(Micro Cloud Foundry Only) Register a new user.  Requires administrator privileges:

```bash
prompt$ vmc add-user --email &lt;newemail@email.com&gt; --passwd &lt;newpasswd&gt;
```

(Micro Cloud Foundry Only) Delete a registered user and all applications and service instances associated with the user.  Requires administrator privileges:

```bash
prompt$ vmc delete-user &lt;useremail@email.com&gt;
```

##Deploying Applications

Deploy an application by executing the command from the directory that contains the application artifacts (such as `myapp.war` or `myapp.rb`).  This is the interactive way to deploy an application, and the VMC command prompts you for information such as the name of the application, its deployment URL, its programming framework, memory allocation, and the service instances that will be bound to it.   The command then deploys the application to the target to which your current session is associated, stages it, and starts it so that it is immediately available.

```bash
prompt$ vmc push
```

You can specify one or all of the following options to pass deployment values;  if you do not specify an option, `vmc push` will interactively prompt you for it:

```bash
prompt$ vmc push &lt;appname&gt; --path &lt;directory&gt; --url &lt;deploymentURL&gt; --instances &lt;instance-number&gt;  --mem &lt;MB&gt; --no-start
```

where:

+ *appname* refers to the internal name you want to give to your application.
+ *directory* refers to the absolute or relative local directory name that contains your application.  Note that *all* files in the directory will be pushed by the `vmc push` command, so be sure you include only the files you want to deploy in the directory.
+ *deploymentURL* refers to the URL that you will later use to invoke your application in a browser; default is `&lt;appname&gt;.cloudfoundry.com`
* *instance-number* specifies the number of instances of the application to start; default is 1
+ *MB* specifies the upper memory limit of the application, in MB; default is 512 MB
+ *--no-start* specifies that you do not want Cloud Foundry to actually start the application yet, which it will do by default.

The `vmc push` command will still prompt you for the runtime and associated service instances, even if you specify all the options above.

For example:

```bash
prompt$ vmc push hello --path /usr/bob/sample-apps/hello --url hello-bob.cloudfoundry.com --instances 2 --mem 64 --no-start
```

##Updating a Deployment

Update a deployed application with the application bits in the current directory:

```bash
prompt$ vmc update &lt;appname&gt;
```

**Note**: Be sure you specify the *name* of the application (first column in the output of `vmc apps`) rather than its deployment URL.

The preceding command immediately ends any existing user sessions connected to the deployed application.

Update the memory (in MB) that has been reserved for an existing deployment; the application will be automatically restarted:

```bash
prompt$ vmc mem &lt;appname&gt; &lt;MB&gt;
```

**Note**: Use `vmc stats &lt;appname&gt;` to view the currently reserved memory for the application, as well as how much it is currently using.

Register a new deployment URL with an existing deployed application; this command *adds* the URL to the existing list of registered URLs:

```bash
prompt$ vmc map &lt;appname&gt; &lt;URL&gt;
```

**Note**: Use `vmc apps` to view the list of URLs currently registered for a deployed application.

Unregister a URL for a deployed application:

```bash
prompt$ vmc unmap &lt;appname&gt; &lt;URL&gt;
```

Scale an application (both up and down) by changing the number of instances that are currently deployed:

```bash
prompt$ vmc instances &lt;appname&gt; &lt;number-of-instances&gt;
```

**Note**: Use `vmc apps` to view the current number of instances for a deployed application.

##Managing the Deployment Lifecycle (Start, Stop, Delete)

Stop a currently running deployment:

```bash
prompt$ vmc stop &lt;appname&gt;
```

**Note**: Be sure you specify the *name* of the application (first column in the output of `vmc apps`) rather than its deployment URL.

Start a currently stopped deployment:

```bash
prompt$ vmc start &lt;appname&gt;
```

Restart a currently running deployment:

```bash
prompt$ vmc restart &lt;appname&gt;
```

Delete an application:

```bash
prompt$ vmc delete &lt;appname&gt;
```

Rename an application:

```bash
prompt$ vmc rename &lt;appname&gt; &lt;new-appname&gt;
```

##Managing and Binding Services

Display the supported service types and the instances of these services you have already provisioned:

```bash
prompt$ vmc services
```

**Note**: The output of the preceding command is split into two tables: the first table displays the available service types that you can use for your applications, the second table displays the services for which you have already created an instance (using the `vmc create-service` command.)  These are called *provisioned services*. Use  the `vmc apps` command to determine which of these service instances are currently bound to deployed applications.

Create a new instance of a service type and assign it a name:

```bash
prompt$ vmc create-service &lt;service-type&gt; &lt;service-instance-name&gt;
```

**Note**: Use `vmc services` to view the list of available service types for which you can create an instance.  Use the name of the service type listed in the first column of the first table.

Create a new instance of a service type, assign it a name, and immediately bind it to a deployed application:

```bash
prompt$ vmc create-service &lt;service-type&gt; &lt;service-instance-name&gt; &lt;appnam&gt;
```

Bind a service instance to a deployed application.   Assumes that you have already deployed the application using `vmc push`:

```bash
prompt$ vmc bind-service &lt;service-instance-name&gt; &lt;appname&gt;
```

**Note**:  Use `vmc services` to view the name of your service instance (first column of second table).

Remove the binding between a service instance and an application:

```bash
prompt$ vmc unbind-service &lt;service-instance-name&gt; &lt;appname&gt;
```

Delete an instance of a service:

```bash
prompt$ vmc delete-service &lt;service-instance-name&gt;
```

To bind an existing service instance at the same time you deploy an application using `vmc push`, enter `y` at the two prompts then specify the number of the service instance from the provided list.  For example (only relevant output and prompts of the `vmc push` command shown):

```bash
prompt$ vmc push
        ...
        Would you like to bind any services to 'pizza-juliet'? [yN]: y
        Would you like to use an existing provisioned service [yN]? y
        The following provisioned services are available:
        1. Insight-19ff7b1
        2. mysql-juliet
        Please select one you wish to provision: 2
        Binding Service: OK
        Uploading Application:
        ...
        Starting Application: OK

```

To create a new service instance at the time you deploy an application, and then bind the new service instance to the application, specify the appropriate prompts and information about the new service instance.  For example (only relevant output and prompts of `vmc push` shown):

```bash
prompt$ vmc push
        ...
        Would you like to bind any services to 'pizza-juliet'? [yN]: y
        Would you like to use an existing provisioned service [yN]? n
        The following system services are available:
        1. mongodb
        2. mysql
        3. postgresql
        4. rabbitmq
        5. redis
        Please select one you wish to provision: 2
        Specify the name of the service [mysql-ab63]: mysql-new
        Creating Service: OK
        Binding Service: OK
        Uploading Application:
        ...
        Starting Application: OK

```

##Getting Information about the Cloud Foundry Target

Display the supported programming languages:

```bash
prompt$ vmc runtimes
```

Display the supported programming frameworks:

```bash
prompt$ vmc frameworks
```

Display the supported service types (such as MySQL and RabbitMQ) and the instances of these service types that you have created:

```bash
prompt$ vmc services
```

**Note**: The output of the preceding command is split into two tables: the first table displays the available service types that you can use for your applications, the second table displays the services for which you have already created an instance (using the `vmc create-service` command.)  Use  the `vmc apps` command to determine which of these service instances are currently bound to deployed applications.

##Getting Information about Applications

Display the list of applications that are currently deployed for your account, along with instances, health, and associated service instances:

```bash
prompt$ vmc apps
```

Display the standard output log entries for an application:

```bash
prompt$ vmc logs &lt;appname&gt;
```

**Note**: Be sure you specify the *name* of the application (first column in the output of `vmc apps`) rather than its deployment URL.

Display the recent crashes of a particular application:

```bash
prompt$ vmc crashes &lt;appname&gt;
```

Display any fatal errors that occurred for an application:

```bash
prompt$ vmc crashlogs &lt;appname&gt;
```

Display resource information (such as core usage, memory, disk space and uptime) of each instance of a deployed application:

```bash
prompt$ vmc stats &lt;appname&gt;
```

List the environment variables for an application:

```bash
prompt$ vmc env &lt;appname&gt;
```

Add an environment variable to an application:

```bash
prompt$ vmc env-add &lt;appname&gt; &lt;variable=value&gt;
```

Delete an environment variable you previously added to an application:

```bash
prompt$ vmc env-del &lt;appname&gt; &lt;variable&gt;
```
