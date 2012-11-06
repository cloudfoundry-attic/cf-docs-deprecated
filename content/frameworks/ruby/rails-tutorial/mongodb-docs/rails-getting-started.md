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

##Understanding How Rails MVC Works

![Image for Rails MVC Architecture](/images/rails-tutorial/rails-mvc.png)

The browser provides the clients view. It sends it's request to the web server which forwards to the dispatcher. The dispatcher loads the controller. The controller may select to access the database via ActiveRecord via simple CRUD commands. The controller then supplies a response, directly via an Action view, or via a delegation to Action Web Services or by delivery to Action Mailer. This is the view that becomes available in the browser.

##Index

Concepts covered in this tutorial are given below

<table class="spring-tutorial-index-table">
	<thead>
  	<tr>
	    <th>Exercise No</th>
	    <th>Exercises</th>
	    <th>Starters</th>
	    <th>Complete</th>
    </tr>
  </thead>
  <tbody>
    <tr>
	    <td>1</td>
	    <td><a href='/frameworks/ruby/rails-tutorial/mongodb-docs/rails-new-template-with-mongodb.html'>New template  Application with MongoDB</a></td>
	    <td></td>
	    <td></td>
    </tr>
    <tr>
      <td>2</td>
      <td><a href='/frameworks/ruby/rails-tutorial/mongodb-docs/rails-user-login.html'>Adding User Authentication to the Application</a></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <td>3</td>
      <td><a href='/frameworks/ruby/rails-tutorial/mongodb-docs/rails-expense-user-flow.html'>Adding Expense User WorkFlow to the Application</a></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <td>4</td>
      <td><a href='/frameworks/ruby/rails-tutorial/mongodb-docs/rails-expense-admin-flow.html'>Adding Expense Admin WorkFlow to the Application</a></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <td>5</td>
      <td><a href='/frameworks/ruby/rails-tutorial/mongodb-docs/rails-hosting-application-with-vmc.html'>Deploying the Application to Cloud Foundry using VMC</a></td>
      <td></td>
      <td></td>
    </tr>
  </tbody>
</table>

<a class="button-plain" style="padding: 3px 15px; float: right" href="/frameworks/ruby/rails-tutorial/mongodb-docs/rails-new-template-with-mongodb.html">Next</a>
