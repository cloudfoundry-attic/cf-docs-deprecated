---
title: Rails Tutorial
description: Ruby on Rails Application Development with Cloud Foundry
tags:
    - tutorial
    - ruby
    - rails
    - vmc
---

##Objective

This is a guide for Ruby developers to create an application using Rails framework and host the application on Cloud Foundry and Micro Cloud Foundry by using VMC - a command line tool for managing applications on Cloud Foundry.

+ Cloud Foundry is an open source cloud computing platform as a service (PaaS) software developed by VMware. Learn more about Cloud Foundry [here](http://www.cloudfoundry.com/about)
+ Micro Cloud Foundry is a downloadable version of Cloud Foundry that can run on a developer’s machine.


##Prerequisites

This guide makes the following assumptions:

+ [Ruby and Rails](frameworks/ruby/installing-ruby.html) have been installed on your system to build rails application.
+ You have a [Cloud Foundry account](https://my.cloudfoundry.com/signup) to host the application on Cloud Foundry.
+ [Micro Cloud Foundry](/infrastructure/micro/installing-mcf.html) have been installed on your machine (optional) to host the applications locally.
+ [vmc](/tools/vmc/installing-vmc.html) have been installed on your system to manage the applications on Cloud Foundry.
+ You are proficient in developing Ruby applications with Rails framework.

For more information about Ruby and Cloud Foundry, see:

+  [Ruby on Rails](http://rubyonrails.org/)
+  [Getting Started with Cloud Foundry](/getting-started.html)

##Overview

Ruby on Rails is an open-source web framework written in Ruby, that is optimized for programmer's happiness and sustainable productivity. It is a dynamically typed programming language similar to Python, Smalltalk, and Perl. It lets you write more beautiful code by favoring Convention over Configuration. It is an MVC (model, view, controller) framework where Rails provides all the layers and they work together seamlessly. Learn more about Ruby on Rails [here](http://rubyonrails.org/) and [documentation](http://guides.rubyonrails.org/).

##Understanding How Rails MVC Works

![Image for Rails MVC Architecture](/images/screenshots/rails/postgres/rails-getting-started/rails-mvc.png)

The browser displays the clients view. It sends it's request to the web server that intern forwards it to the dispatcher. The dispatcher loads the controller. The controller may select to access the database via ActiveRecord via simple CRUD commands. The controller then supplies a response, directly via an Action view, or via a delegation to Action Web Services or by delivery to Action Mailer. This is the view that becomes available in the browser.

##Next Steps

+ [Rails App with PostgreSQL](/frameworks/ruby/rails-tutorial/rails-app-with-postgres.html)
+ [Rails App with MongoDB](/frameworks/ruby/rails-tutorial/rails-app-with-mongodb.html)