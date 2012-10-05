---
title: Expense Reporting Application with Rails
description: Deploy the Application on Cloud Foundry with VMC
tags:
    - ruby
    - rails
    - vmc
---

##Introduction

This is the continuation for the previous [chapter](/frameworks/ruby/rails-tutorial/mongodb-docs/rails-expense-admin-flow.html). This section clearly explains hoe to host and manage the applications on Cloud Foundry using `vmc` a command line tool for hosting the applications on Cloud Foundry.

##Prerequisites

+ Should have completed [chapter-4](/frameworks/ruby/rails-tutorial/rails-expense-admin-flow.html) or must have an application that can push to Cloud Foundry.
+ [Cloud Foundry account](https://my.cloudfoundry.com/signup) to host the application on Cloud Foundry.
+ [Micro Cloud Foundry](/infrastructure/micro/installing-mcf.html) installed on your machine `(optional)` to host the applications locally.
+ [vmc](/tools/vmc/installing-vmc.html) installed on your system to manage the applications on Cloud Foundry.

##Deploy the Application on Cloud Foundry with VMC

Pushing an application to Cloud Foundry is very simple. You first need to creat an account [here](https://my.cloudfoundry.com/signup) inorder to do the same.

*vmc* is a gem that has the feature to push the application to Cloud Foundry through command line. To use it, install the gem by following command.

```bash
$ gem install vmc
```

The following command will show you a list of all commands available in vmc.

```bash
$ vmc -h
```

Set your target to a particular domain by following the command. Here the target has been set to `api.cloudfoundry.com`. It can also target to micro cloud foundry if you want to host applications on your micro-cloud-foundry locally.

```bash
$ vmc target api.cloudfoundry.com
Successfully targeted to [http://api.cloudfoundry.com]
```

Now login using the account details given in the confirmation mail using the following command.

```bash
$ vmc login
```

You will be asked for username and password. Specify the email and password used to register the account in Cloud Foundry. Once you login successfully for the first time, change the password. You can do this by executing the following command. This will change the passord for current logged-in user.

```bash
$ vmc passwd
```

Once you make changes for authorization, its time for push the application to Cloud Foundry. Use the following command to see a list of all applications hosted on Cloud Foundry.

```bash
$ vmc apps
```

and we can see a list of all runtimes available on Cloud Foundry by following command.

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

Deploying an application to Cloud Foundry is as simple as executing one command.

##Deploying Application with Rails versions below 3.1

There are two ruby versions availabe on Cloud Foundry, Ruby-1.8.7 and Ruby-1.9.2p180. You can push the applications using `vmc push`.

```bash
$ vmc push
```

Version 1.8.7 will be considered by default. Therefore if we wish to push an application with Ruby-1.9.2, we must specify it explicitely as mentioned below.

```bash
$ vmc push --runtime ruby19
```

When you push the application like explained above, you will be asked for some options required to host the application.

##Deploying Application with Rails 3.1 or above versions

Before pushing any Rails 3.1 applications or above, we need to do the following things.

###Bundle Your packages.

Packages needs to be bundled before pushing the application to Cloud Foundry.

```bash
$ bundle package

$ bundle install
```

This will compress your gems and package into `vendor/cache` folder, so those gem files are served in Cloud Foundry.

##Changing Configurations

By default serving static assets are disabled in production environment. To enable this configuration, open your `config/environments/production.rb` file and make the changes mentioned below.

```bash
config.serve_static_assets = false
```
to

```bash
config.serve_static_assets = true
```

##Precompiling Assets

Assets Precompile is a new property of rails 3.1 or above, where every assets like `css, javascripts and images` are compiled into `public/assets` and will be server in side the application. Do that by executing the following command.

```bash
$ bundle exec rake assets:precompile
```

##Commit to VCS

If you're using a Version Controle System, commit the current configuration to your version control system. Consider including `Gemfile.lock` and gems packaged into `vendor/cache` and assets compiled into `public/assets`

Now you are ready to push the application to Cloud Foundry. Push the application as shown below.

```bash
$ vmc push --runtime ruby19
```

Respond to some interactive prompts to push the application on Cloud Foundry. These will be as shown below.

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
+-------------------+----+---------+-------------------------------------+------------------+

```

Open your browser and visit the given url. Here it is [expense-reporting.cloudfoundry.com](http://expense-reporting.cloudfoundry.com)

##Seeding the database

Use the `vmc rails-console` to populate the database once your application is deployed on Cloud Foundry. Note that memory used while running the console contributes to your overall application memory limit.

`rails-console` in vmc requires `caldecott` gem to be installed in your system. Just install the gem by following this line in terminal

```bash
$ gem install caldecott
```

Now open `rails-console` for the application `expense_reporting` by running the following command. This will open a console like a local rails-console which will ge opend in local by running `rails c`. Here you can query the database entities like a normal rails console.

```bash
$ vmc rails-console expense-reporting
```

When starting the console for the first time, it deploys 'caldecott' application to the Cloud Foundry and provides the functionality for managing the services in Cloud Foundry. As soon as the *caldecott* application is hosted on the Cloud Foundry, *irb* will be opened to use the interactive ruby command line tool. Populate the data from `db/seeds.rb` file like below.

<pre class="terminal">
$ vmc rails-console app-name
Connecting to 'app-name' console: OK

irb():001:0>`rake db:seed`
This will execute the seeds.rb file
</pre>