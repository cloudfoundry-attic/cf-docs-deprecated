---
title: Expense Reporting Application with Rails
description: Creating a new template with MongoDB
tags:
    - ruby
    - rails
    - template
    - mongodb
---

##Introduction

This section introduces the Expense Reporting application, teaches you how to get started with a new Rails application with MongoDB and some other required configurations  for the application.

##Prerequisites

+ [Ruby and Rails](frameworks/ruby/installing-ruby.html) installed on your machine.
+ [MongoDB](http://www.mongodb.org/display/DOCS/Quickstart) installed on your machine.

Starting a new application in Rails is as simple as executing a single command `rails new <appname>`. So before we start on the new application, let us go through the application overview.

##Subtopics

+ [Expense Reporting Application Overview](#expense-reporting-application-overview)
+ [Creating a new Template with MongoDB](#creating-new-template-with-mongodb)
+ [Configuring the Application](#configuring-the-application)
+ [Schema Design](#schema-design)

##Expense Reporting Application Overview

This application is a sample demo application which will clearly explain about expense reporting.

High-Level overview of Expense Reporting Processing:

In the Expense Reporting application, we shall assign two roles -

1. Normal User: Can create an expense, view its status, edit or delete it
2. Admin User: Can Approve or Reject expenses that are created

once the Normal User submits the expense report, it is routed to Admin User for his final decision on it.

In short,

+ Normal user will be able to see all expense-reports created by him along with their status, He allowed to create new expense report at any time.
+ Admin user will be able to see the list all expense reports to be approved and also he can create and see list of his expenses.

##Creating New Template with MongoDB

As we know, running an application in Ruby on Rails depends on the versions of Ruby and Rails in which the application has been developed. So we first decide the versions of both Ruby and Rails.

For now, CloudFoundry supports for only two versions of Ruby ruby1.8.7 and ruby1.9.2-p180. This tutorial chooses to use the newer of the two versions. And for rails, the latest available version, and Rails 3.2.8.

Install Ruby through `rvm`, a version management for ruby. And choose the Ruby version by following this command.

```bash
$ rvm use 1.9.2-p180
```

`rvm` offeres a good mechanism called `gemset` for handling gems and their dependencies for particular Ruby versions. Create a separate gemset for the Expense Reporting Application.

```bash
$ rvm gemset use expense_reporting --create
```

This command will create the expense_reporting gemset if it isn't existing and in use already. Now, install Rails gem to be able to genarate a new rails application. Use the following command for installing it

```bash
$ gem install rails
```
This will install the latest version of Rails, rails-3.2.8 in this case.

This tutorial chooses MongoDB as the database. By default Rails will generate a new application with `sqlite` database, and there is no option to get started with MongoDB. Hence we need to generate the new application by skipping active-record as mentioned below

```bash
$ rails new expense_reporting --skip-active-record
```

This will create a new application and also finish bundle installation. Next, go to the application folder.

```bash
$ cd expense_reporting
```

##Check Point

That is it. Start the server by following the command.

```bash
$ rails s
```

The server runs on port 3000. So open your browser and go to the url [localhost:3000](http://localhost:3000). The Index page will now visible

![Image for the index page](/images/rails-tutorial/rails-welcome.png)

##Configuring the application

Creating a file with name .rvmrc will tell the rvm to use a particular Ruby version and gemset whenever you enter the application folder through the terminal. Place a file named .rvmrc in the main folder and write the following line in.

```ruby
rvm use 1.9.2-p180@expense_reporting --create
```

###Removing the default index.html page

Remove the `public/index.html` page which will be rendered when you start the application in browser.

```bash
$ rm public/index.html
```

###Gemfile

Bundler is a tool that maintains a consistent environment for a developer’s code across all machines. Know more about bundler [here](http://gembundler.com/).  Gemfile is the place where all the gems that are used in the application can be placed. Bundler will install all the gems that are mentioned in the Gemfile along with their dependencies. You will find the `Gemfile` in the root folder of the aplication.

[Mongoid](https://github.com/mongoid/mongoid) is an ODM (Object-Document-Mapper) framework for [MongoDB](http://www.mongodb.org/) written in Ruby. The philosophy of Mongoid is to provide a familiar API to Ruby developers who have been using ActiveRecord or Data Mapper, while leveraging the power of MongoDB's schemaless and performant document-based design, dynamic queries, and atomic modifier operations. Learn more about [Mongoid here](http://mongoid.org/en/mongoid/)

In the Gemfile, add the gems `mongoid` and `bson_ext` to use MongoDB and replace the `jquery-rails` gem with `cloudfoundry-jquery-rails`. Now the Gemfile will look like below

```ruby
source 'https://rubygems.org'

gem 'rails', '3.2.8'

gem "mongoid", "~> 2.4"
gem 'bson_ext'

group :assets do
  gem 'sass-rails',   '~> 3.2.3'
  gem 'coffee-rails', '~> 3.2.1'
  gem 'uglifier', '>= 1.0.3'
end

# gem 'therubyracer'

# gem 'jquery-rails'
gem 'cloudfoundry-jquery-rails'
```

Use the gem `therubyracer` if you do not have javascript environment in your system. This will be useful for the assets. To install javascript runtime, install node.js by following [these instructions](http://howtonode.org/how-to-install-nodejs) for instructions

Install all the gems by running the following command.

```bash
$ bundle install
```

##Configuring the database

Before configuring the database in the application, make sure that you already installed MongoDB in your local machine and the mongo server is up. You can [follow this](http://www.mongodb.org/display/DOCS/Quickstart) to install MongoDB.

Now we need to generate the configuration file which will be used to keep the settings for configuring MongoDB database. Do this by executing the following command

```bash
$ rails g mongoid:config
    create  config/mongoid.yml
```

This will create `config/mongoid.yml` file with default settings. The file looks like below.

```ruby
development:
  host: localhost
  database: expense_reporting_development

test:
  host: localhost
  database: expense_reporting_test

# set these environment variables on your prod server
production:
  host: <%= ENV['MONGOID_HOST'] %>
  port: <%= ENV['MONGOID_PORT'] %>
  username: <%= ENV['MONGOID_USERNAME'] %>
  password: <%= ENV['MONGOID_PASSWORD'] %>
  database: <%= ENV['MONGOID_DATABASE'] %>
  # slaves:
  #   - host: slave1.local
  #     port: 27018
  #   - host: slave2.local
  #     port: 27019
```

Unlike Relational Databases, there is no need to create databases explicitly in MongoDB. As the data is stored in documents, it will create necessary databases and documents whenever it needed.

##Adding Styles to the Application

Here we have a customised UI for this Expense Reporting application. To use these styles, add the css file into your `app/assets/stylesheets` folder. [Get the the CSS files from here](/rails-code/stylesheets.zip)

After adding the css files, [get the images from here](/rails-code/ui-images.zip) and place them in your `app/assets/images` folder.

##Schema Design

As explained in the overview, the application will deal with one User model and one Expense model. Each user can have many expenses and each expense belongs to a User as well as to an ExpenseType.

```ruby
class User
  has_many :expenses, :dependent => :destroy
end

class Expense
  belongs_to :user
  belongs_to :expense_type
end

class ExpenseType
  has_many :expenses
end
```

<a class="button-plain" style="padding: 3px 15px;" href="/frameworks/ruby/rails-tutorial/mongodb-docs/rails-getting-started.html">Prev</a>  <a class="button-plain" style="padding: 3px 15px; float: right;" href="/frameworks/ruby/rails-tutorial/mongodb-docs/rails-user-login.html">Next</a>
