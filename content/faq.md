---
title: FAQ
description: Frequently Asked Questions
tags:
    - faq
---

### How Do I Get Started on Cloud Foundry
For resources on getting started with Cloud Foundry, please refer to our [Getting Started Guide](/getting-started.html).

### How can I change my CloudFoundry.com password?

Please use the following link to reset your CloudFoundry.com password: [https://my.cloudfoundry.com/passwd](https://my.cloudfoundry.com/passwd)

### I'm having trouble with my login credentials

Here are a few things you can do to make sure that you are using your credentials correctly:

+ Verify that your target is set correctly ($vmc target api.cloudfoundry.com)
+ Check the target ($vmc target and $vmc info)
+ Verify that you are using the vmc client ($vmc login [email] [password] )

Please try with the temporary password provided in the Welcome email from Cloud Foundry after sign-up.  If you had subsequently changed that with a 'vmc passwd' command, please try with that password also.

If you are still not able to use vmc, open a ticket by clicking on the "Submit a request" tab. Make sure to provide us with the details and the support team will look into it for you.

### Why can't I register the application name I want?

Application names must be longer than 3 characters. In addition, please note that some names are reserved and will not be allowed even if they are of the right length.

If you receive the message: "The URI: 'fo.cloudfoundry.com' has already been taken or reserved", please pick another name for your application.


### Can apps use the local file system for storage?
Your application needs to treat local file storage as ephemeral, which can disappear when your application stops, crashes, or moves. It should not be used for durable storage of content such as user-uploaded files. Further, if more than one instance of your app is running the local storage is only visible to a specific instance of the app, and is not visible or shared across all instances. If persistent storage is required, a local service such as [MongoDB GridFS](http://www.mongodb.org/display/DOCS/GridFS) or external blob stores such as [Box.net](http://developers.blog.box.com/2011/12/14/deploying-to-cloud-foundry-and-heroku-from-box/) or [Amazon S3](http://aws.amazon.com/s3/) can be used.

### Is there session support?
When scaling an app to multiple instances the topic of session management arises. Cloud Foundry uses session affinity, also known as sticky sessions: when a user login request is handled by a particular instance of your app, future requests from that user/session will be routed to the same app instance. Sticky sessions are provided for all apps and do not have to be configured, with the exception of Node.js. Node.js apps need to set a cookie named JSESSIONID to a unique value for each user/session.

Cloud Foundry does not offer session replication: if the particular instance crashes the user is routed to another instance which won't have authentication data in the session, and the user will have to log in again.

### What happens when a service runs out of capacity?
When a service runs out of capacity (e.g. MySQL hits the 128MB size limit), the service switches to a read/delete-only mode. For example, on a SQL service removing data and reading data is allowed, but writing more data is disabled:

```bash
delete from table where ... # allowed
drop table ... # allowed
select ... from ... # allowed
insert into ... # not allowed
update ... # not allowed
```

The application bound to a service that has run out of capacity may see errors such as "INSERT is disabled." Other services (redis, MongoDB) will exhibit similar behavior.

### Can I send and receive email from my app?
Applications cannot send outbound email via TCP port 25. An app can use an external email service available on a different port (such as 587 for SMTP-AUTH). Applications cannot receive email by listening on port 25 either. We recommend using an external mail service such as [SendGrid](http://sendgrid.com/) or [Mailgun](http://mailgun.net/).

### Is there a firewall that prevents my app from accessing external services?
No, your application has direct access to the internet with the exception of connecting to TCP port 25 (used for sending email).

### Why is my app returning "504 Gateway Time-out" errors?
The error page originates from the CloudFoundry.com HTTP Router and not your application. Application requests taking longer than 30 seconds are terminated by the Router. The 30-second timeout cannot be adjusted and applies to all apps running on CloudFoundry.com. However, the Router supports long-polling and streaming on a rolling 30-second window: if an application sends data to a client within 30 seconds, the timeout counter is reset and starts again at 30 seconds. To troubleshoot 504 errors, please investigate what is causing the delayed response from an app and ensure data is returned every 30 seconds.


### I'm having issues with HTTP_proxy

When CloudFoundry.com launched in April 2011, applications making outbound requests had to use an HTTP proxy (for about the first 10 days of operation). When we removed the need for a proxy, we did not eliminate the HTTP_PROXY, http_proxy, etc. environment variables. These variables used to be made available to your application, however, now we are eliminating the HTTP_PROXY, HTTPS_PROXY, http_proxy, https_proxy, no_proxy, NO_PROXY. For the vast majority of applications, this change will have no effect. All well written applications either use http client classes that correctly process and use proxy related environment variables, OR the application writer codes his algorithms to correctly use the proxy environment variables when they are present.

However, some applications may  be hard-coded to look up one or all of the following proxy-related environment variables and either use the values incorrectly, OR not correctly handle the absence of an environment variable. The variables:

```bash
- http_proxy

- HTTP_PROXY

- https_proxy

- HTTPS_PROXY

- no_proxy

- NO_PROXY
```

As of May 14th, 2012 these environment variables will not be defined for apps on CloudFoundry.com. If your app is not working as expected, please check your code for any references to the above variables and remove them. The safest method would be searching for any references to proxy in your application root:

```bash
$ cd myapp
$ grep -i -r proxy .
```

### What runtimes are supported on CloudFoundry.com (beta)?

Use the vmc runtimes command to view the complete list of supported runtimes ($vmc runtimes)

### What are the account, application and service limits for my CloudFoundry.com (beta) account?

**Account Limits**

Limits associated with a user account in CloudFoundry.com (beta) can be obtained using the following command.

```bash
$ vmc info

VMware's Cloud Application Platform
For support visit support@cloudfoundry.com

Target:   http://api.cloudfoundry.com (v0.999)
Client:   v0.3.10

User:     user@email.com
Usage:    Memory   (256.0M of 2.0G total)
          Services (1 of 16 total)
          Apps     (2 of 20 total)
```

For accounts:
Applications are limited to 20.
Service instances are limited to 16.
Memory is limited to 2GB.


**Application Limits**

For each application:
Memory is limited to 2GB.
Disk space is limited to 2GB.
CPU	is limited to the fair share of 4 cores.
Network	is limited to your fair share.
File descriptors are limited to	256.
URIs are limited to 4.

Total Number of Instances: Limited by the amount of reservable memory, 2GB.  For example, if you reserve 128M for an application, you can have upto 16 instances, 256M -> 8 instances, 512M -> 4 instances and so on.  (assuming just one application).

Also, per-application usage statistics are available using the following command:


```bash
$ vmc stats <appname>

+----------+-------------+----------------+--------------+--------------+
| Instance | CPU (Cores) | Memory (limit) | Disk (limit) | Uptime       |
+----------+-------------+----------------+--------------+--------------+
| 0        | 0.0% (1)    | 19.4M (128M)   | 14.5M (2G)   | 0d:2h:13m:6s |
+----------+-------------+----------------+--------------+--------------+
```

Memory : 19.4M used, 128M reserved.  ($ vmc mem appname command can be used to update the memory reservation, please note that this will restart the application)

Disk : 14.M used of 2G limit

**Service Limits**

For services:
MySQL DB size is limited to	128MB.
Redis memory is limited to 16MB.
MongoDB memory is limited to 240MB.



### My app stopped running.

If any application stops running without the end user initiating that action, we consider it a crash.

When an application crashes, vcloudlabs will maintain the crashed state for a period of time for introspection.

You can see this state via **vmc crashlogs <appname>**

In the case of an app running successfully and then crashing, it could be something app specific or it could be that the application is using more resources than it is allowed.


**vmc info**


This will show you the account wide resources for memory, services, and application instances. This is a good place to check first.


**vmc stats \<appname\>**


This will show you detailed information on the resource usage of your application. So for instance, the application that runs [http://www.cloudfoundry.com](http://www.cloudfoundry.com/), named **www**, outputs the following:


**~> vmc stats www**

Instance   CPU (Cores)     Memory (limit)  Disk (limit)    Uptime
---------  -----------     --------------  ------------    ------
0          0.0% (8)        29.2M (64M)     824.0K (2G)     60d:19h:53m:58s
1          0.0% (8)        29.3M (64M)     816.0K (2G)     60d:19h:50m:3s


This will show you various resource usage levels and their current limits.


Currently CloudFoundry.com will terminate applications that exceed their memory or disk limits. This information will be available through the crashlogs command to vmc as outlined above.


If your app is stopping unexpectedly after running for some time, and you don't suspect your app, please use this checklist to see if you are running into a resource limit issue.


### I'm getting the VMC Error: 'The input stream is exhausted'

**Problem**

On some Mac OSX releases the stock ruby installed causes vmc client tool to fail emitting following error.  This is due to a library (used by vmc) incompatibility with the packaged ruby that comes with OSX.

For example in Mac OSX Lion, the stock ruby is 1.8.7 with patchlevel 249

```bash
$ ruby --version
ruby 1.8.7 (2010-01-10 patchlevel 249) [universal-darwin11.0]
```

And the error shows up during vmc transaction as

```bash
$ vmc push
Would you like to deploy from the current directory? [Yn]: The input stream is exhausted.
/Library/Ruby/Gems/1.8/gems/highline-1.6.2/lib/highline.rb:608:in `get_line'
/Library/Ruby/Gems/1.8/gems/highline-1.6.2/lib/highline.rb:629:in `get_response'
/Library/Ruby/Gems/1.8/gems/highline-1.6.2/lib/highline.rb:216:in `ask'
/Library/Ruby/Gems/1.8/gems/vmc-0.3.12/lib/cli/commands/apps.rb:370:in `push'
/Library/Ruby/Gems/1.8/gems/vmc-0.3.12/lib/cli/runner.rb:428:in `send'
/Library/Ruby/Gems/1.8/gems/vmc-0.3.12/lib/cli/runner.rb:428:in `run'
/Library/Ruby/Gems/1.8/gems/vmc-0.3.12/lib/cli/runner.rb:14:in `run'
/Library/Ruby/Gems/1.8/gems/vmc-0.3.12/bin/vmc:5
/usr/bin/vmc:19:in `load'
/usr/bin/vmc:19
```

**Solution**

Solution to this problem is to install [RVM](https://rvm.beginrescueend.com/) (Ruby Version Manager) and install and use 1.8.7 (or 1.9.2).  Typically higher patchlevels will get installed.  But please note that even if you install the same patchlevel as the stock ruby the problem will be solved.

A quick guide to installing RVM:
1.  Install rvm ([http://beginrescueend.com/rvm/install/](http://beginrescueend.com/rvm/install/))
2.  Add the below lines to ~/.bash_profile file
```bash
# Ruby Version Manager
[[ -s "$HOME/.rvm/scripts/rvm" ]] && . "$HOME/.rvm/scripts/rvm"
```
3.  Open new terminal
4.  Re-install ruby 1.9.2 using rvm and also re-install the vmc gem under rvm installed ruby
```bash
$ rvm install 1.9.2
$ rvm use 1.9.2
$ gem install vmc
```


**Sample Use**

Here is a transcript using rvm installed ruby.

```bash
$ rvm use 1.8.7-p249
Using /Users/Alex/.rvm/gems/ruby-1.8.7-p249
```
```bash
$ ruby --version
ruby 1.8.7 (2010-01-10 patchlevel 249) [i686-darwin11.0.0]
```
```bash
$ vmc push
Would you like to deploy from the current directory? [Yn]:
Application Name: hello-sinatra
Application Deployed URL: 'hello-sinatra.cloudfoundry.com'? hello-sinatra-alexsuraci.cloudfoundry.com
Detected a Sinatra Application, is this correct? [Yn]:
Memory Reservation [Default:128M] (64M, 128M, 256M, 512M or 1G)
Creating Application: OK
Would you like to bind any services to 'hello-sinatra'? [yN]:
Uploading Application:
  Checking for available resources: OK
  Packing application: OK
  Uploading (2K): OK
Push Status: OK
Staging Application: OK
Starting Application: OK
```


### Can I use Hyperic to monitor my applications on Cloud Foundry?

[Yes](http://blog.cloudfoundry.com/2011/06/29/hyperic-brings-application-monitoring-to-cloud-foundry/)


### My Ruby/Sinatra/Rails app is failing on start.

When performing a vmc push for a Ruby/Sinatra/Rails app, and after answering all the prompted questions, you might see this error:

Starting Application: ............Error: Application [APP]'s state
is undetermined, not enough information available.

The reason for this error is likely a missing gem or gems. Depending on where the issue occurred, additional details are typically found in one of 4 logs: staging.log, migration.log, stderr.log, and stdout.log

To diagnose: use "vmc files app-name logs" to view the log files, and then "vmc files app-name logs/stderr.log" to view a log.

In Cloud Foundry, we require that all gem dependencies are formally called out. For a simple Sinatra app, a simple Gemfile like this (in the root directory) is the typical fix:

Gemfile contains:

```bash
source "http://rubygems.org"
gem 'bundler'
gem 'sinatra'
gem 'json'
gem 'httpclient'
gem 'redis'
```

With this in place, run "bundle package; bundle install" to correct the problem

If you can not find the missing elements for upload, see if you can find the answer in the forums or create your own question thread. Alternatively send us an e-mail to support.cloudfoundry.com and we will try to help.

One of the common causes for the sample demo applications (hello.rb) as shown in our demos failing to start is due to copy/paste, which copies in smart quotes and causes the application to not run.  Please type in the demo application by hand.  Also, please make sure that you do not have any extraneous tab (\t) characters in the source file, and replace them with spaces.


### Is there a Getting Started guide in Japanese?

[Yes](/getting-started-japanese.html)

### How can I update my application without dropping user traffic?

In general, when developing and testing applications with Cloud Foundry, **vmc update** fits the bill quite well.

However, when an app is considered "live", like our own www site, you should follow the steps below that will ensure no user requests are drop during the transition.

We use the concept of a new application that will assume the URL space of the original application.

Here is how to update your application without any downtime:

NOTE: 'vmc update' will drop traffic..

- vmc push [app]NEW (bind any shared services, like DB, Cache, etc)

 TEST [app]NEW.cloufoundry.com

- vmc map [app]NEW [app].cloudfoundry.com

 TEST [app].cloudfoundry.com several times, looping through new and old now..

- vmc unmap [app]OLD [app].cloudfoundry.com # Does not drop traffic, just kills all new traffic

 TEST

- vmc delete [app]OLD


### My application is still reporting as running even after exiting.

If you exit an app using System.exit(), Tomcat's security manager will prevent the process from terminating. This will cause the system to report the application as running. We will be adding a custom health check so that app terminated in this way may be detected by the system.
