---
title: Expense Reporting Application with Rails
description: Deploy the Application on Cloud Foundry with VMC
tags:
    - ruby
    - rails
    - vmc
---

##Introduction

This is the continuation for the previous [chapter](/frameworks/ruby/rails-tutorial/rails-expense-admin-flow.html). This section clearly explains hoe to host and manage the applications on Cloud Foundry using `VMC` a command line tool for hosting the applications on Cloud Foundry.

##Prerequisites

+ You should complete the [chapter-4](/frameworks/ruby/rails-tutorial/rails-expense-admin-flow.html) to complete the application or else you should have some application to push that to Cloud Foundry.
+ [Cloud Foundry account](https://my.cloudfoundry.com/signup) to host the application on Cloud Foundry.
+ [Micro Cloud Foundry](/infrastructure/micro/installing-mcf.html) installed on your machine `(optional)` to host the applications locally.
+ [VMC](/tools/vmc/installing-vmc.html) installed on your system to manage the applications on Cloud Foundry.

##Deploy the Application on Cloud Foundry with VMC

It is very simple to push an application to Cloud Foundry. The first thing we need for pushing the application to Cloud Foundry is need to create an account [here](https://my.cloudfoundry.com/signup).

*VMC* is a gem which will provide feature to push the application to Cloud Foundry through command line. For using this, we need to install the gem by following command.

```bash
$ gem install vmc
```

The folloing command will gives you a complete list of commands that are available in vmc.

```bash
$ vmc -h
```

Set your target to particular domain by following command. Here i am setting the target to cloudfoundry. And we can also target to our micro cloud foundry if we want to host applications on micro-cloud-foundry in locally.

```bash
$ vmc target api.cloudfoundry.com
Successfully targeted to [http://api.cloudfoundry.com]
```

Once we successfully able to target to the cloud-foundry, we need to login to our particular account with the details given by the cloud-foundry in confirmation mail. We can login by following command.

```bash
$ vmc login
```

This will ask for username and password. You have to specify the email and password with which you have registered the account in cloud foundry. After login successfully for the first time, change the password. You can do so by executing the following command. This will change the passord for current logged-in user.

```bash
$ vmc passwd
```

Once you make changes for authorization, now its time for play with pushing the application to Cloud Foundry. We can see list of all applications hosted on Cloud Foundry by following command.

```bash
$ vmc apps
```

and we can see the list of all runtimes available on cloudfoundry by following command.

```bash
$ vmc runtimes

+--------+-------------+-----------+
| Name   | Description | Version   |
+--------+-------------+-----------+
| java   | Java 6      | 1.6       |
| java7  | Java 7      | 1.7       |
| ruby18 | Ruby 1.8    | 1.8.7     |
| ruby19 | Ruby 1.9    | 1.9.2p180 |
| node   | Node.js     | 0.4.12    |
| node06 | Node.js     | 0.6.8     |
| node08 | Node.js     | 0.8.2     |
+--------+-------------+-----------+
```

##Deploying application

Its very simple to deploy the applications to Cloud Foundry, just by one command we can do so. 
##Deploying Application with Rails versions below 3.1

There are two ruby versions availabe on Cloud Foundry and they are Ruby-1.8.7 and Ruby-1.9.2p180. We can push the applications by `vmc push`.

```bash
$ vmc push
```

By default this will consider the Ruby version as 1.8.7. So if we want to push an application with Ruby-1.9.2, we have to specify that explicitely as mentioned below.

```bash
$ vmc push --runtime ruby19
```

This will consider the Ruby 1.9.2-p180 version. And when you push the application as mentioned above, it will ask you for some options required to host the application.

##Deploying Application with Rails 3.1 or above versions

Before pushing any Rails 3.1 applications or above, we need to do some thing.

###Bundle Your packages.

We need to bundle the packages before pushing the application to Cloud Foundry.

```bash
$ bundle package

$ bundle install
```

This will compress your gems and package into `vendor/cache` folder, so that those gem files will be served in Cloud Foundry.

##Changing Configurations

By default in production environment, serving static assets will be disabled. We need to make this configuration enable. So open your `config/environments/production.rb` file and do the change as mentioned below.

```bash
config.serve_static_assets = false
```
to

```bash
config.serve_static_assets = true
```

##Precompiling Assets

Assets Precompile is a new property of rails 3.1 or above, where it will compile every assets like `css, javascripts and images` into `public/assets`. So that those will be server in side the application. So this will be done by the following command.

```bash
$ bundle exec rake assets:precompile
```

##Commit to VCS

If you are using any Version Controle System, commit the current configuration to your version control system. Consider including `Gemfile.lock` and gems packaged into `vendor/cache` and assets compiled into `public/assets`

Now we are ready to push the application to Cloud Foundry. Just push the application as shown below. Here it is for ruby version 1.9.2p180

```bash
$ vmc push --runtime ruby19
```

Respond to some Interactive prompts to push the application on Cloud Foundry. These will be as shown below.

```bash
$ vmc push --runtime ruby19
Would you like to deploy from the current directory? [Yn]: Y
Application Name: expense-reporting
Detected a Rails Application, is this correct? [Yn]: Y
Application Deployed URL [expense-reporting.cloudfoundry.com]: 
Memory reservation (128M, 256M, 512M, 1G, 2G) [256M]: 
How many instances? [1]: 
Bind existing services to 'test-exp-rep'? [yN]: N
Create services to bind to 'test-exp-rep'? [yN]: y
1: mongodb
2: mysql
3: postgresql
4: rabbitmq
5: redis
What kind of service?: 3
Specify the name of the service [postgresql-82e81]: exp_rep_db
Create another? [yN]: N
Would you like to save this configuration? [yN]: y
Manifest written to manifest.yml.
Creating Application: OK
Creating Service [exp_rep_db]: OK
Binding Service [exp_rep_db]: OK
Uploading Application:
  Checking for available resources: OK
  Processing resources: OK
  Packing application: OK
  Uploading (762K): OK   
Push Status: OK
Staging Application 'exp_rep_db': OK                                          
Starting Application 'exp_rep_db': OK
```

Now you can check your apps list hosted on Cloud Foundry.

```bash
$ vmc apps

+-------------------+----+---------+-------------------------------------+------------------+
| Application       | #  | Health  | URLS                                | Services         |
+--------------+----+---------+------------------------------------------+------------------+
| expense-reporting | 1  | RUNNING | expense-reporting.cloudfoundry.com  | exp_rep_db       |
+-------------------+----+---------+----------------    -----------------+------------------+

```

Now open browser and visit the url mentioned over there. Here it is [expense-reporting.cloudfoundry.com](http://expense-reporting.cloudfoundry.com)