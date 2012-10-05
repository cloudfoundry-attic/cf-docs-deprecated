---
title: Expense Report Application with Rails and PostgreSQL
description: Creating a new template with PostgreSQL
tags:
    - ruby
    - rails
    - template
---

##Introduction

This section introduces the Expense Report application, teaches you how to get started with a new Rails application with PostgrSQL and some other required configurations for the application.

##Prerequisites

+ [Ruby and Rails](frameworks/ruby/installing-ruby.html) installed on your machine.
+ [PostgreSQL](http://www.postgresql.org/download/) installed on your machine.

Starting a new application in Rails is as simple as executing a single command `rails new <appname>`. So before we start on the new application, let us go through the application overview.

##Subtopics

+ [Expense Report Application Overview](#expense-report-application-overview)
+ [Creating a new Template with Postgres](#creating-new-template-with-postgresql)
+ [Configuring the Application](#configuring-the-application)
+ [Schema Design](#schema-design)

##Expense Report Application Overview

This application is a sample demo application which will clearly explain about expense report.

High-Level overview of Expense Report Processing:

In the Expense Report application, we shall assign two roles -

1. Normal User: Can create an expense, view its status, edit or delete it
2. Admin User: Can Approve or Reject expenses that are created

once the Normal User submits the expense report, it is routed to Admin User for his final decision on it.

In short,

+ Normal user will be able to see all expense-reports created by him along with their status, He allowed to create new expense report at any time.
+ Admin user will be able to see the list all expense reports to be approved and also he can create and see a list of his expenses.

##Creating New Template with PostgreSQL

As we know, running an application in Ruby on Rails depends on the versions of Ruby and Rails in which the application has been developed. So we first decide the versions of both Ruby and Rails.

For now, CloudFoundry supports for only two versions of Ruby ruby1.8.7 and ruby1.9.2-p180. This tutorial chooses to use the newer of the two versions. And for rails, the latest available version, and Rails 3.2.9.

Install Ruby through `rvm`, a version management for ruby. And choose the Ruby version by following this command.

```bash
$ rvm use 1.9.2-p180
```

`rvm` offeres a good mechanism called `gemset` for handling gems and their dependencies for particular Ruby versions. Create a separate gemset for the Expense Report Application.

```bash
$ rvm gemset use expense_report --create
```

This command will create the expense_report gemset if it isn't existing and in use already. Now, install Rails gem to be able to genarate a new rails application. Use the following command for installing it

```bash
$ gem install rails
```
This will install the latest version of Rails, rails-3.2.9 in this case.

This tutorial chooses PostgreSQL as the database. Generate the new application with postgreSQL database by providing the option as `-d postgresql`.

```bash
$ rails new rails-expense-report-postgres -d postgresql
```

This will create a new application and also finish bundle installation. Next, go to the application folder.

```bash
$ cd rails-expense-report-postgres
```

##Check Point

That is it. Start the server by following the command.

```bash
$ rails s
```

The server runs on port 3000. So open your browser and go to the url [localhost:3000](http://localhost:3000). The Index page will now visible

![Image for the index page](/images/screenshots/rails/postgres/rails-new-template-with-postgres/rails-welcome.png)

##Configuring the application

Creating a file with name .rvmrc will tell the rvm to use a particular Ruby version and gemset whenever you enter the application folder through the terminal. Place a file named .rvmrc in the main folder and write the following line in.

```ruby
rvm use 1.9.2-p180@expense_report --create
```

###Removing the default index.html page

Remove the `public/index.html` page which will be rendered when you start the application in browser.

```bash
$ rm public/index.html
```

###Gemfile

Bundler is a tool that maintains a consistent environment for a developer’s code across all machines. Know more about bundler [here](http://gembundler.com/).  Gemfile is the place where all the gems that are used in the application can be placed. Bundler will install all the gems that are mentioned in the Gemfile along with their dependencies. In the gem file, replace the `jquery-rails` gem with `cloudfoundry-jquery-rails`. Now the Gemfile will look like below

```ruby
source 'https://rubygems.org'

gem 'rails', '3.2.9'
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

Use the gem `therubyracer` if you do not have javascript environment in your system. This will be useful for the assets. To install javascript runtime, install node.js by following [these instructions](http://howtonode.org/how-to-install-nodejs) for instructions

Install all the gems by running the following command.

```bash
$ bundle install
```

##Configuring the database

Before configuring the database in the application, make sure that you already installed PostgreSQL in your local machine. You can [follow this](http://www.postgresql.org/download/) to install PostgreSQL.

`config/database.yml` is the file, where Rails looks for database authentications. This file contains the authentications for accessing the PostgreSQL db in the system and also tells the application which database to use in specific environment. Here the db can be configured for different environments. Update your PostgreSQL username and password credentials in database.yml file.

```ruby
development:
  adapter: postgresql
  encoding: unicode
  database: rails-expense-report-postgres_development
  pool: 5
  username: <username>
  password: <password>
  host: localhost

test:
  adapter: postgresql
  encoding: unicode
  database: rails-expense-report-postgres_test
  pool: 5
  username: <username>
  password: <password>
  host: localhost

production:
  adapter: postgresql
  encoding: unicode
  database: rails-expense-report-postgres_production
  pool: 5
  username: <username>
  password: <password>
  host: localhost
```

Let's create the databases mentioned in the above file by running the following command.

```bash
$ bundle exec rake db:create
```
##Adding Styles to the Application

Here we have a customised UI for this Expense Report application. To use these styles, add the css file into your `app/assets/stylesheets` folder. [Get the the CSS files from here](/rails-code/stylesheets.zip)

After adding the css files, [get the images from here](/rails-code/ui-images.zip) and place them in your `app/assets/images` folder.

##Schema Design

Postgresql uses ActiveRecord for the query interface. ActiveRecord connects business objects and database tables to create a persistable domain model where logic and data are present in one wrapping. It is an implementation of the object-relational mapping (ORM) pattern. ActiveRecord provides automated mapping between classes and tables, attributes and columns. Learn more about ActiveRecord [here](http://ar.rubyonrails.org/).

As explained in the overview, the application will deal with one User model and one Expense model. Each user can have many expenses and each expense belongs to a User as well as to an ExpenseType.

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

<a class="button-plain" style="padding: 3px 15px;" href="/frameworks/ruby/rails-tutorial/rails-getting-started.html">Prev</a>  <a class="button-plain" style="padding: 3px 15px; float: right;" href="/frameworks/ruby/rails-tutorial/postgres/rails-user-login.html">Next</a>
