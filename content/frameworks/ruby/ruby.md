---
title: Ruby
description: Ruby Application Development with Cloud Foundry
tags:
    - ruby
    - overview
---

This guide for Ruby developers describes Cloud Foundry support for Ruby
applications, including applications built using the Sinatra or Ruby on Rails
frameworks.

This guide makes the following assumptions:

-  You have [installed Ruby and Ruby Gems](installing-ruby.html)
-  You have [installed vmc](/tools/vmc/installing-vmc.html)
-  You have a [Cloud Foundry account](http://www.cloudfoundry.com/signup)
-  You are proficient in developing Ruby applications with your framework of
   choice

Subtopics:

-  [Ruby Versions](#supported-ruby-versions)
-  [Gemfile and Gem Support](#gemfile-and-gem-support)
-  [Using Cloud Foundry Services with Ruby](#using-cloud-foundry-services)
-  [What to do Next](#what-to-do-next)

## Supported Ruby Versions

As of this writing, Cloud Foundry supports `ruby 1.8.7-p302` or higher and
`ruby 1.9.2-p180` or higher.
Ruby 1.8.7 is the default. If you use Ruby 1.9.2, add the vmc `--runtime ruby19`
option when you push your application to Cloud Foundry:

    vmc push <appname> --runtime ruby19

## Gemfile and Gem Support

Cloud Foundry requires a Gemfile in the root of your application that lists
the gems your application depends upon.

You should use [Bundler](http://gembundler.com) to package your applications.
Run `bundle install;bundle package` each time you modify your Gemfile and
before any `vmc push` or `vmc update` command.

For more details see:

+ [Ruby on Cloud Foundry](/frameworks/ruby/ruby-cf.html)

## Using Ruby on Rails with Cloud Foundry

Ruby on Rails works well with Cloud Foundry. There are some differences in the
requirements for Rails 3.0 and 3.1 or 3.2 which you can read about at these links:

+   [Ruby on Rails 3.0](/frameworks/ruby/rails-3-0.html)
+   [Ruby on Rails >= 3.1](/frameworks/ruby/rails-3-1.html)

## Using Cloud Foundry Services

You can use any of the following Cloud Foundry services in your Ruby application:

-   [MySQL](http://www.mysql.com/), the open source relational database
-   [MongoDB](http://www.mongodb.org/), the scalable, open,
    document-based database
-   [vFabric Postgres](http://www.vmware.com/products/datacenter-virtualization/vfabric-data-director),
    the VMware vFabric Postgres 9.0 relational database
-   [RabbitMQ](http://www.rabbitmq.com/), for messaging
-   [Redis](http://redis.io/), the open source key-value data structure server

While you develop your application, you can run these services locally, or you
can use [Micro Cloud Foundry](/infrastructure/micro/installing-mcf.html) to host your application.
An advantage of using Micro Cloud Foundry is that your application runs in an
environment that is very similar to Cloud Foundry. Also, with Micro Cloud
Foundry, you do not have to set up local services, and you do not have to tool
your application to connect to different services for your development and
deployment environments.

Following are examples and tutorials for using the Cloud Foundry services with Ruby and Ruby frameworks.

-   [MySQL](/services/mysql/ruby-mysql.html)
-   [MongoDB](/services/mongodb/ruby-mongodb.html)
-   [MongoDB with GridFS](/services/mongodb/ruby-mongodb-gridfs.html)
-   [RabbitMQ](/services/rabbitmq/ruby-rabbitmq.html)

## What to do Next

-   [VMC Installation](/tools/vmc/installing-vmc.html) describes how to install the Cloud Foundry command-line interface. If you have Ruby installed, it is as simple as installing the vmc gem.
-   [VMC Quick Reference](/tools/vmc/vmc-quick-ref.html)
-   [Creating a Simple Ruby Application](ruby-simple.html) is a tutorial that shows you how to create a simple Sinatra application and deploy it to Cloud Foundry.
-   [Ruby on Rails 3.0 Development with Cloud Foundry](rails-3-0.html) provides the information you need to successfully deploy Rails 3.0 applications on Cloud Foundry.
-   [Ruby on Rails 3.1 Development with Cloud Foundry](rails-3-1.html) provides the information you need to successfully deploy Rails 3.1 applications on Cloud Foundry.