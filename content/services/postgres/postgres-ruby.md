---
title: PostgreSQL, Ruby
description: PostgreSQL for Cloud Foundry - Rails Tutorial
tags:
    - postgres
    - ruby
    - rails
    - tutorial
---

This tutorial explains how to use the PostgreSQL service for Cloud Foundry from a Rails 3 application.

In this tutorial, we'll build a very simple application that uses PostgreSQL service. Once you understand that, you will be able to integrate more realistic uses of the service into your Rails 3 applications on Cloud Foundry.

This tutorial goes through the whole process of creating a simple Rails 3 application on Cloud Foundry. The application manages a basic guestbook page. Every visitor can sign the guestbook by filling a simple form. All visitors that have already signed are listed with date.

## Prerequisites

+ Ruby 1.8 or 1.9
+ Rubygems
+ VMC command line tool
+ Rails 3

To start off, there are a few things that need to be installed on your development machine. We assume that you already have a recent version of Ruby 1.8 or 1.9 installed, and rubygems too.

Install Rails via rubygems:

```bash
$ gem install rails
[...]
Successfully installed rails-3.0.9
```


Install VMC command line tool:

```bash
$ gem install vmc
Fetching: vmc-0.3.12.gem (100%)
Successfully installed vmc-0.3.12
1 gem installed
```


## Create a Rails 3 application

Now we will create a Rails 3 application. We will use rails to generate a new Rails 3 application to use as our start point.

**$ rails new guestbook --database=postgresql**

create

create README

create Rakefile

[...]

**$ cd guestbook**

To implement the logic, we need one table "Entry" with two columns: one for guest name, the other for last update time. Thanks to Rails 3 framework, now we can use scaffold command to generate model, views, controller and migration for the table "Entry".

**$ rails generate scaffold Entry name:string**

We don't need to specify the last update time explicitly, because Rails 3 framework will generate "updated_at" column by default.

Edit file app/views/entries/index.html.erb to add one line (highlighted in red color below) to show the last update time for each entry.


![aurora.png](http://support.cloudfoundry.com/attachments/token/wauy78kuctleafi/?name=aurora.png)


Now the application is ready to be pushed to Cloud Foundry.



## Deploy Application

Set vmc to target the cloudfoundry.com site, and log in:

```bash
$ vmc target api.cloudfoundry.com
Succesfully targeted to [http://api.cloudfoundry.com]
```

```bash
$ vmc login
Email: [your Cloud Foundry email]
Password: [your Cloud Foundry password]
Successfully logged into [http://api.cloudfoundry.com]
```


Push application:

```bash
$ vmc push
Would you like to deploy from the current directory? [Yn]: Y
Application Name: [your application name]
Application Deployed URL: '[Your application name].cloudfoundry.com'? Y
Detected a Rails Application, is this correct? [Yn]: Y
Memory Reservation [Default:256M] (64M, 128M, 256M or 512M) 64M
Creating Application: OK
Would you like to bind any services to 'guestbook-rails'? [yN]: Y
Would you like to use an existing provisioned service [yN]? N
The following system services are available:
1. mongodb
2. mysql
3. postgresql
4. rabbitmq
5. redis
Please select one you wish to provision: 3
Specify the name of the service [postgresql-1c51b]: [your database service name]
Creating Service: OK
Binding Service: OK
Uploading Application:
 Checking for available resources: OK
 Processing resources: OK
 Packing application: OK
 Uploading (5K): Push Status: OK
Staging Application: ...
Staging Application: OK
Starting Application: OK
```

Now you can go to URL http://your application name].cloudfoundry.com/entries to see the page and play with it.