---
title: Expense Reporting Application with Rails
description: Creating new template with Postgresql
tags:
    - ruby
    - rails
    - template
---

##Introduction

This section provides the details of what is the Expense Reporting application, how to get started with new Rails application and some other required configurations  for the application. 

##Prerequisites

+ [Ruby and Rails]() installed on your machine.
+ [Postgresql]() installed on your machine.

Starting a new application in rails is as simple as just executing one command `rails new <appname>` So before starting new application, will go through the application overview.

##Subtopics

+ [Expense Reporting Application Overview](#expense-reporting-application-overview)
+ [Creating new Template with Postgres](#creating-new-template-with-postgres)
+ [Configuring Appication](#configuring-application)
+ [Schema Design](#schema-design)

##Expense Reporting Application Overview

This application is a sample demo application which will clearly explain about expense report. 

High-Level Overview of Expense Report Processing: 

Here in this Application, There are two kinds of roles. They are
 
1. Normal User: who will create the expense-report along with an attachment represents that expense.
2. Admin User: who will Approve or Reject the expense-report created by the normal user.

once the normal user submits the expense-report, it is routed to Admin user for approval. The admin user can accept that expense-report or reject based on his decision.

On the whole, 

+ Normal user will be able to see all expense-reports created by him along with their status, and at any time he is allowed to create new expense-report.

+ Admin user will be able to see the list all expense-reports to be approved and also other reports which are approved or rejected previously.

##Creating New Template With Postgres

As we know, running an application in Ruby on Rails basically depends on the versions of Ruby and Rails in which we develop the application. So first we have to decide the versions of both ruby and rails.

For now, CloudFoundry is providing the support for only two versions of ruby. they are ruby1.8.7 and ruby1.9.2-p180. So i am going to choose the new one among the available versions. And for rails, the latest available version Rails 3.2.8.

Hope you have installed the ruby through `RVM`, a version management for ruby. So choose the ruby version by following command.

```bash
$ rvm use 1.9.2-p180
```

RVM provides a good mechanism called `gemset` for handling the gems and their dependencies for particular ruby version. So create a separate gemset for our Expense Reporting Application.

```bash
$ rvm gemset use expense_reporting --create
```

this will create the expense_reporting gemset if it does not exist already, and uses it. Now, install rails gem, so that we will be able to genarate a new rails application. use the following command for installing rails gem

```bash
$ gem install rails
```
This will install the latest version of rails, here it is rails 3.2.8. Now, start the new application. Here i am going to choose the database as Postgresql. so generating the new application with postgresql database by providing the option as `-d postgresql`.

```bash
$ rails new expense_reporting -d postgresql
```

this will create a new application and also will do bundle install by default. Then go to that application's folder.

```bash
$ cd expense_reporting
```

Thats it. Now start the server by following command. Now we can see the index page.

```bash
$ rails s
```

The server will run on port 3000. So open browser and go to the url `localhost:3000`. We will see the default index page from `/public/index.html` which will be rendered by default in rails.

![Image for the index page](/images/rails-default-index.png)

##Configuring Application

Creating .rvmrc file will be used to tell the rvm to use a particular ruby version and gemset whenever you entered into the application folder through terminal. Hence create a file with name `.rvmrc`.Place it in the main folder and write the following line in that file

```ruby
rvm use 1.9.2-p180@expense_reporting --create
```

###Removing the default index.html page

Basically rails will render the default `index.html` page from `public` folder every time you open the application in browser. To get rid of this, remove the index.html page.

```bash
$ rm public/index.html
```

###Gemfile

Bundler is a tool that maintains a consistent environment for a developer’s code across all machines. Know more about bundler [here](http://gembundler.com/).  Gemfile is the place where we can place all the gems that are used in the application. Bundler will install all the gems that are mentioned in the Gemfile and their dependencies. In the gem file, replace the `jquery-rails` gem with `cloudfoundry-jquery-rails`. Now the Gemfile looks like below

```ruby
source 'https://rubygems.org'

gem 'rails', '3.2.8'
gem 'pg'

group :assets do
  gem 'sass-rails',   '~> 3.2.3'
  gem 'coffee-rails', '~> 3.2.1'
  gem 'uglifier', '>= 1.0.3'
end

# gem 'therubyracer'

# gem 'jquery-rails'
gem 'cloudfoundry-jquery-rails'
```

Use the gem `therubyracer` if you do not have javascript environment in your system. This will be useful for the assets. To install javascript runtime, you have to install node.js, you can follow [this](http://howtonode.org/how-to-install-nodejs) for instructions

install all the gems by run the following command.

```bash
$ bundle install
```

##Configuring Database

`/config/database.yml` is the file, where rails looks for the database authentications. This file contains the authentications for accessing the postgresql db in system and also tells the application which database to use for particular environment. Here we can configure the db for different environments. So Please update your postgresql username and password credentials in database.yml file. Here in the below file the username is `postgres` and there is no password in my case.

```ruby
development:
  adapter: postgresql
  encoding: unicode
  database: expense_reporting_development
  pool: 5
  username: postgres
  password:
  host: localhost

test:
  adapter: postgresql
  encoding: unicode
  database: expense_reporting_test
  pool: 5
  username: postgres
  password:
  host: localhost

production:
  adapter: postgresql
  encoding: unicode
  database: expense_reporting_production
  pool: 5
  username: postgres
  password:
  host: localhost
```

Now we need to create the databases mentioned in the above file. We can do this by running following command.

```bash
$ bundle exec rake db:create	
```

##Schema Design

Postgresql will use ActiveRecord for query interface. Active Record connects business objects and database tables to create a persistable domain model where logic and data are present in one wrapping. It‘s an implementation of the object-relational mapping (ORM) pattern. ActiveRecord provides Automated mapping between classes and tables, attributes and columns. Learn more about Active Record [here](http://ar.rubyonrails.org/).

As explained in the application overview, the application is going to deal with one User model and also one model for Expense. Each user can have many expenses whereas each expense belongs to a User.Also, each expense belongs to ExpenseType.

```ruby
class User < ActiveRecord::Base
  has_many :expenses, :dependent => :destroy
end

class Expense < ActiveRecord::Base
  belongs_to :user
  belongs_to :expense_type
end

class ExpenseType < ActiveRecord::Base
  has_many :expenses
end
```

[<= Prev](/frameworks/ruby/rails-tutorial/rails-getting-started.html)  <span style="float: right;">[Next =>](/frameworks/ruby/rails-tutorial/rails-user-login.html)</span>
