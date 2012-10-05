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

This is a guide for ruby developers to successfully create an application using rails framework and host the application on Cloud Foundry and Micro Cloud Foundry by using VMC - a command line tool for managing the applications on Cloud Foundry.

+ Cloud Foundry is an open source cloud computing platform as a service (PaaS) software developed by VMware. [Learn more about Cloud Foundry](http://www.cloudfoundry.com/about)
+ Micro Cloud Foundry is a downloadable version of Cloud Foundry that can run on a developer’s machine.


##Prerequisites

This guide makes the following assumptions:

+ [Ruby and Rails](frameworks/ruby/installing-ruby.html) installed on your system to build rails application.
+ [Cloud Foundry account](https://my.cloudfoundry.com/signup) to host the application on Cloud Foundry.
+ [Micro Cloud Foundry](/infrastructure/micro/installing-mcf.html) installed on your machine (optional) to host the applications locally.
+ [VMC](/tools/vmc/installing-vmc.html) installed on your system to manage the applications on Cloud Foundry.
+ You are proficient in developing ruby applications with rails framework.

For more information about Ruby and Cloud Foundry, see:

+  [Ruby on Rails](http://rubyonrails.org/)
+  [Getting Started with Cloud Foundry](/getting-started.html)

##Overview

Ruby on Rails is an open-source web framework written in Ruby, that is optimized for programmer's happiness and sustainable productivity. It is a dynamically typed programming language similar to Python, Smalltalk, and Perl. It lets you write more beautiful code by favoring Convention over Configuration. It is an MVC (model, view, controller) framework where Rails provides all the layers and they work together seamlessly. Learn more about [Ruby on Rails](http://rubyonrails.org/) and [documentation](http://guides.rubyonrails.org/).

+ [Creating new template for ExpenseReport application with Postgres](/frameworks/ruby/rails-tutorial/rails-new-template-with-postgres.html)
+ [Adding User Login to ExpenseReport Application](/frameworks/ruby/rails-tutorial/rails-user-login.html)
+ [Adding Expenses to the Application](/frameworks/ruby/rails-tutorial/rails-expense-user-flow.html)
+ [Adding Admin WorkFlow to the Application](/frameworks/ruby/rails-tutorial/rails-expense-admin-flow.html)
+ [Hosting application with VMC](/frameworks/ruby/rails-tutorial/rails-hosting-application-with-vmc.html)

<a class="button-plain" style="padding: 3px 15px; float: right" href="/frameworks/ruby/rails-tutorial/rails-new-template-with-postgres.html">Next</a>
