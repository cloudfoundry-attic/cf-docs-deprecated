---
title: VMC Installation
description: Installing the Command-Line Interface (vmc)
tags:
    - vmc
    - install
    - CLI
---

You use the Cloud Foundry command-line interface (known as `vmc`) at a Unix terminal or Windows command prompt to execute all the Cloud Foundry operations, such as configuring your applications and deploying them to Cloud Foundry.

You execute the `vmc` commands in the same way whether you are deploying your application to the PaaS Cloud Foundry (`cloudfoundry.com`), or to your own local version of Cloud Foundry (Micro Cloud Foundry).  The basic commands are the same; the only difference is that you initially specify a different target before you log in using your Cloud Foundry credentials.

This section describes the prerequisites for installing `vmc`, installation instructions, and how to deploy a simple application.

**Subtopics**

+ [Prerequisite: Installing Ruby and RubyGems](#prerequisite-installing-ruby-and-rubygems)
+ [Installing vmc: Procedure](#installing-vmc-procedure)
+ [Verifying the Installation by Deploying a Sample Application](#verifying-the-installation-by-deploying-a-sample-application)
+ [Next Steps](#next-steps)

## Prerequisite: Installing Ruby and RubyGems

`vmc` is delivered as a Ruby gem, which means you must install Ruby and RubyGems (a Ruby package manager) on the computer on which you want to run `vmc`, if you have not already done so.

The following versions of Ruby are currently supported:

* 1.8.7
* 1.9.2

If you have already installed Ruby and RubyGems, then you can skip to [Installing vmc: Main Steps](#installing-vmc-main-steps).

The following sections provide basic information about installing Ruby and RubyGems on Windows and a variety of Linux computers:

+ [Windows](/frameworks/ruby/installing-ruby.html#windows)
+ [Mac OS X](/frameworks/ruby/installing-ruby.html#mac-os-x)
+ [Ubuntu](/frameworks/ruby/installing-ruby.html#ubuntu)
+ [Redhat/Fedora](/frameworks/ruby/installing-ruby.html#redhatfedora)
+ [Centos](/frameworks/ruby/installing-ruby.html#centos)
+ [SuSE](/frameworks/ruby/installing-ruby.html#suse)
+ [Debian](/frameworks/ruby/installing-ruby.html#debian)

## Installing vmc: Procedure

Installing `vmc` is easy once you have installed [Ruby and RubyGems](#prerequisite-installing-ruby-and-rubygems) on your computer.

*  If you haven't already done so, signup for your free [Cloud Foundry](http://cloudfoundry.com/) account.  You will receive an email with your user credentials.

* Open a terminal (Linux) and execute the following command:

```bash
prompt$ sudo gem install vmc
```

   Consult your system administrator for any required authentication credentials for the `sudo` command.
   On Windows, open a command prompt in which Ruby is enabled and execute the following:

```bash
prompt> gem install vmc
```

* Execute the `vmc target` command to specify the Cloud Foundry target to which you will deploy your applications:

    + To deploy on the PaaS Cloud Foundry, specify `api.cloudfoundry.com`
    + To deploy on your local Micro Cloud Foundry, specify `api.<appname>.cloudfoundry.me`, where *appname* is the domain you registered for your application at the Micro Cloud Foundry Web site.  See [Installing Micro Cloud Foundry](/infrastructure/micro/installing-mcf.html).

    The following command targets the PaaS Cloud Foundry:

```bash
prompt$ vmc target api.cloudfoundry.com
```

    To determine your current target, execute the `vmc target` command without any parameters:

```bash
prompt$ vmc target
```

*   Login using the user credentials you received via email after you registered with Cloud Foudnry.  Your username is typically your email address.

```bash
prompt$ vmc login
```

*  Ensure you have successfully logged in by retrieving information about your account:

```bash
prompt$ vmc info
```

*  Change your password:

```bash
prompt$ vmc passwd
```

*  View the full list of VMC commands, along with their parameters and a brief description, by executing the `vmc help` command:

```bash
prompt$ vmc help
```

You have now successfully installed `vmc` and run a few basic commands.

## Verifying the Installation by Deploying a Sample Application

Now that you have installed VMC and logged in to your target, you can start deploying applications to the Cloud.

This section shows how to deploy a simple application that does not require any services (such as MySQL or RabbitMQ).  The purpose of the section is for you to quickly get a feel for VMC and Cloud Foundry by deploying and running a very basic application.  Later sections describe how to configure your application to use services that connect to databases or manage messaging.

* Create a simple application that does not require any services and package it appropriately, such as into a `*.war` file for Spring applications.

    If you do not currently have an application, see [Creating a Simple Sinatra Application](#creating-a-simple-sinatra-application) for instructions on how to create a basic Ruby Hello World application using Sinatra in just a few minutes.

*  Open a terminal window (Linux) or command prompt (Windows) and change the directory that contains your application.

    For example, if you created the simple Ruby [Hello World](#creating-a-simple-sinatra-application) application using Sintatra:

```bash
prompt$ cd /usr/bob/sample-apps/hello
```

* Deploy your application using the `vmc push` command, which interactively prompts for deployment information:

```bash
prompt$ vmc push
```

For prompts that require a yes or no answer, the default value is shown using capital letters.  For example, if "yes" is the default, you'll see `[Yn]`.

The following sample output also shows the responses you should provide; for clarity, default values are specified explicitly.  See further explanations about these prompts after the example:

```bash

   Would you like to deploy from the current directory? [Yn] Yes
   Application Name: hello
   Application Deployed URL: 'hello.cloudfoundry.com'?  hello-bob.cloudfoundry.com
   Detected a Sinatra Application, is this correct? [Yn]  Yes
   Memory Reservation [Default:128M]  (64M, 128M, 256M, 512M or 1G) (Press Enter to take default)
   Would you like to bind any services to 'hello'? [yN]: No

```

After you complete the prompts, `vmc` provides the following output for a successful push (deployment):

```bash

     Uploading Application:
       Checking for available resources: OK
       Packing application: OK
     Uploading (0K): OK
     Push Status: OK
     Staging Application: OK
     Starting Application: OK

```

The Application Name refers to the internal name of the application as well as the actual file you want to deploy without its file extension, `hello` in our example.  The Application Deployed URL is the URL that you use in your browser to run the application after it has successfully deployed and started on Cloud Foundry.  Be sure that you specify a unique deployment URL, or `vmc` will return an error message that the URI has already been taken or reserved.  In the example above, the URL is `hello-bob.cloudfoundry.com`.

Verify that your application is available by executing the `vmc apps` command:

```bash
$ vmc apps

    +--------------+----+--------+-------------------------------+----------+
    | Application  | #  | Health | URLS                          | Services |
    +--------------+----+--------+-------------------------------+----------+
    | hello        | 1  | RUNNING| hello-bob.cloudfoundry.com    |          |
    +--------------+----+--------+-------------------------------+----------+

```

Run your application in your browser by going to the URL you provided to the `vmc push` command, which in the example above is `hello-bob.cloudfoundry.com`.

If, for example, you deployed the Hello World Sinatra application, you should see the text `Hello from Cloud Foundry` in your browser.

![hello-bob-app.png](/images/screenshots/installing-vmc/hello-bob-app.png)

## Updating the Deployment

Now that you have your first application deployed, it is easy to update if you make changes to it, as describe in the following procedure.

Change your application in some way so that, when you run it, you will know which version it is.

For example, in the simple Hello World Sinatra application contained in `hello.rb`, change the text `Hello from Cloud Foundry` to `Hello from Cloud Foundry and VMware`.

At your command prompt or terminal, be sure you are still located in the directory that contains your application file (`/usr/bob/sample-apps/hello.rb` in our example) and execute the `vmc update` command, specifying the name of your application, which in our example is `hello`:

```bash
$ vmc update hello

    Uploading Application:
          Checking for available resources: OK
          Packing application: OK
          Uploading (0K): OK
    Push Status: OK
    Stopping Application: OK
    Staging Application: OK
    Starting Application: OK

```

In your browser, refresh your application and you will see your changes:

![hello-bob-app-updated.png](/images/screenshots/installing-vmc/hello-bob-app-updated.png)

## Creating a Simple Sinatra Application

If you have not already done so, download and install the [Sinatra Web framework](http://www.sinatrarb.com/) on your computer.

Create the directory in which the new application will live.  For example:

```bash
prompt$ mkdir /usr/bob/sample-apps/hello
```

Using your favorite text editor, create a file called `hello.rb` in this new directory with the following contents:

``` ruby
require 'sinatra'
get '/' do
  "Hello from Cloud Foundry"
end
```

![hello.rb](/images/screenshots/installing-vmc/vmc_hello.jpg "hello app")

## Next Steps

+ [Installing Micro Cloud Foundry](/infrastructure/micro/installing-mcf.html)
+ [Deploying and Managing Applications](/tools/deploying-apps.html)
+ [Configuring Applications to Use Cloud Foundry](/frameworks.html)
+ [VMC Quick Reference Guide](/tools/vmc/vmc-quick-ref.html)
+ [Debugging](/tools/vmc/debugging.html)
