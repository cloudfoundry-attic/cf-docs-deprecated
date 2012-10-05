---
title: Expense Report Application with Rails and PostgreSQL
description: Adding User Authentication to Application
tags:
    - ruby
    - rails
    - user_login
---

##Introduction

This chapter is a continuation of [chapter 1](/frameworks/ruby/rails-tutorial/rails-new-template-with-postgres.html). This section explains the process of adding user authentication to the Expense Report Application, so a user can access only the authorized content in the application.

##Prerequisites

+ Should have completed [chapter 1](/frameworks/ruby/rails-tutorial/rails-new-template-with-postgres.html), that explains creating a basic template, configuring the database etc.
+ Should know how to use a gem in a Rails application

##Setup: Exercise2-Starter

If you are starting this chapter from the downloaded source code [Exercise2-Starter.zip](/rails-code/expense-report-postgres/Exercise2-Starter.zip), please setup the application as described [here](/frameworks/ruby/rails-tutorial/psql-starters-guide.html) before continue with this tutorial.

##Adding User Authentication to the application

## **Step 1:** Adding Gem for Authentication

Authentication is a basic need for any application. There are several authentication methods and a lot of gems available on [github](https://github.com/). `Devise` is one of the best gem to use for the same. List of gems for rails authentication can be found at [ruby-toolbox.com](https://www.ruby-toolbox.com/categories/rails_authentication).

Expense Report is an application with basic authentication requirements. A simple authentication gem **sorcery** provides common authentication needs such as signing in/out, activating by email and resetting password. Explore the complete features of this gem at [Wiki for Sorcery](https://github.com/NoamB/sorcery/wiki).

Add the gem to your gem list in Gemfile. Do this by adding the below line to your `Gemfile`. You can find the file  `/Gemfile` in the root folder of the application.

```ruby
gem 'sorcery'
```

Install the gem by running the following command.

```bash
$ bundle install
```

## **Step 2:** Adding User Model

The sorcery gem can now be used in the application. Run the following command to generate the files required to configure the gem.

```bash
$ rails generate sorcery:install
    create  config/initializers/sorcery.rb
    generate  model User --skip-migration
      invoke  active_record
      create    app/models/user.rb
      invoke    test_unit
      create      test/unit/user_test.rb
      create      test/fixtures/users.yml
      insert  app/models/user.rb
      create  db/migrate/20121113091959_sorcery_core.rb
```

As demonstrated, this will generate the initializer file for configuring the settings, User model class without migration file, unit test stubs, and sorcery_core migration file for adding the User objects.

To enable Sorcery authentication in the User model, Sorcery generates the User model with the following line. This adds a number of methods to the User model to handle authentication.

```ruby
class User < ActiveRecord::Base
  authenticates_with_sorcery!
end
```

Sorcery can authenticate based on either username or email, and it will generate a migration file with both email and username fields. Since in this application we shall be authenticating using username, we shall remove the `email` field from the sorcery_core migration file. Here it is `db/migrate/20121113091959_sorcery_core.rb`. The number in the file name `20121113091959` is based on the time at which the file is generated.

```ruby
class SorceryCore < ActiveRecord::Migration
  def self.up
    create_table :users do |t|
      t.string :username,         :null => false  # if you use another field as a username, for example email, you can safely remove this field.
      # t.string :email,            :default => nil # if you use this field as a username, you might want to make it :null => false.
      t.string :crypted_password, :default => nil
      t.string :salt,             :default => nil

      t.timestamps
    end
  end

  def self.down
    drop_table :users
  end
end
```

The Sorcery gem will generate all these fields by default and offers a lot of options for customization. If you wish to change the names of the fields let Sorcery know about the modifications. You will see a list of all options in `config/initializers/sorcery.rb` file.

This application is not handling the functionalities like `Remember me` or `Reset Password`, if you wish to include them, required migration files needs to be generated and Sub modules needs to be added in tha Sorcery configuratoin file. Follow these instructions for [Remember me](https://github.com/NoamB/sorcery/wiki/Remember-Me) and [Reset Password](https://github.com/NoamB/sorcery/wiki/Reset-password)

**Note:** To add a sub module, `Remember_me` for example, the wiki pages start with `rails g sorcery:install remember_me`. This will generate the configuration file for Sorcery and reset all the modifications if any. As we have generated the configuration file already, only the required migration files need to be generated again. To generate just the mgration files, execute `rails g sorcery:install remember_me --migrations`. Then follow the steps in the wiki.

Once you have modified the Sorcery configuration file and migration files accordingly, setup the database by following this command.

```bash
$ bundle exec rake db:migrate
```

## **Step 3:** Adding Validations to User Model

You can add validations or protect any attributes for User model attributes if you wish to. Our User model doesn’t have password or password_confirmation attributes but we’ll be making accessible methods inside the User model to handle them.

```ruby
class User < ActiveRecord::Base
  authenticates_with_sorcery!

  attr_accessible :username, :password, :password_confirmation

  validates_confirmation_of :password
  validates_presence_of :password, :on => :create
  validates_presence_of :username
  validates_uniqueness_of :username
end
```

## **Step 4:** Adding Controllers for Signup

User model is ready, and now generate some controllers to go with it. First create a UsersController to handle the sign-up process.

```bash
$ rails g controller users new
```

Now write the code for `new` and `create` actions in the UsersController. Copy the following code snippet into the file `app/controllers/users_controller.rb`

```ruby
class UsersController < ApplicationController
  before_filter :require_login, :only => [:show]

  def new
    @user = User.new
  end

  def create
    @user = User.new(params[:user])
    if @user.save
      redirect_to root_url, :notice => "Signed up!"
    else
      render :new
    end
  end
end
```

## **Step 5:** Adding View Part for Signup

This is fairly standard Controller code. In the `new` action we create a new User object and in the `create` action we create a new User based on the parameters that are passed in (these will come from the sign-up form) . If the New User is valid, we shall redirect him to the home page (which we don’t have yet), otherwise we render the new action template with error messages. Here we have written `before_filter :require_login`, this method `require_login` is a helper method provided by the sorcery gem that will allow user to perform particular actions only if logged in.

Copy the following snippet of code into `app/views/users/new.html.erb` file. This code generates a form consisting of `email, password and password_confirmation` fields along with some code for displaying validation errors if any.

```rhtml
<div class="signing-form">
  <%= form_for @user do |f| %>
    <% if @user.errors.any? %>
      <div class="error_messages signing-error">
        <ul>
          <% for message in @user.errors.full_messages %>
            <li><%= message %></li>
          <% end %>
        </ul>
      </div>
    <% end %>
    <div>
      <label>Username</label>
      <%= f.text_field :username, :class => "textbox-a", :placeholder => "UserName" %>
    </div>
    <div>
      <label>Password</label>
      <%= f.password_field :password, :class => "textbox-a", :placeholder => "Password" %>
    </div>
    <div>
      <label>Password Confirmation</label>
      <%= f.password_field :password_confirmation, :class => "textbox-a", :placeholder => "Password Confirmation" %>
    </div>
    <div>
      <%= f.submit "Sign Up", :class => "btn-a" %>
    </div>
  <% end %>
</div>
```

## **Step 6:** Adding Login Functionality

Users can Sign up but can’t Signin yet. Fix that by creating a new controller called `sessions_controller` that will handle login and logout.

```bash
$ rails g controller sessions new
```

Use the default implementation of `new` action in `app/cntrollers/sessions_controller.rb` and move on to the view part for this action. In `app/views/sessions/new.html.erb` file we shall create the form for signing in. Hence the following snippet of code should go into `app/views/sessions/new.html.erb`

```rhtml

<div class="signing-form">
  <% if @error_message %>
    <div class="error_messages signing-error">
      <ul>
       <li><%= @error_message %></li>
      </ul>
    </div>
  <% end %>

  <%= form_tag sessions_path do %>
    <div>
      <label>Username</label>
      <%= text_field_tag :username, params[:username], :class => "textbox-a", :placeholder => "UserName" %>
    </div>
    <div>
      <label>Password</label>
      <%= password_field_tag :password, '', :class => "textbox-a", :placeholder => "Password" %>
    </div>
    <div>
      <%= submit_tag "Log in", :class => "btn-a" %>
    </div>
  <% end %>
</div>
```

Here we used [form_tag](http://apidock.com/rails/ActionView/Helpers/FormTagHelper/form_tag) rather than [form_for](http://apidock.com/rails/ActionView/Helpers/FormHelper/form_for), as form_for implies that there’s a resource behind the form. The form submit the data to sessions_path(), which will be the SessionsController’s `create` action. The form has two fields, one each for username and password.

Now write a `create` action to process the login form. Sorcery's Login method can take three parameters: a username or email address, a password that was entered and the value of the remember_me checkbox. This method authenticates and return a User if a matching one is found. We can check for this and redirect back to the home page if a user is found. If not it will display the login form with error message. Copy the following code snippet into `app/controllers/sessions_controller.rb`

```ruby
class SessionsController < ApplicationController
  def new
  end

  def create
    user = login(params[:username], params[:password], params[:remember_me])
    if user
      redirect_back_or_to root_url, :notice => "Logged in!"
    else
      @error_message = "Invalid Username or Password."
      render 'new'
    end
  end
end
```

Instead of using `redirect_to` to redirect back to the home page when a user is found we will use a method that Sorcery gives us called `redirect_back_or_to`. This behaves similar to redirect_to but if a URL is stored by Sorcery it will redirect back to that URL rather than to the one specified in the code. This is useful because it means that if we require a user to log in when they visit a certain page, Sorcery will take them to the login page and then return them back to the page they tried to visit once they had successfully logged in.

Once the signin successful, the User should be redirect_to his expenses index page. As the expenses flow is not implemented yet, let us redirect the User to `show` action in users controller. Modify the `create` method in `app/controllers/sessions_controller.rb` file accordingly.

```ruby
def create
  user = login(params[:username], params[:password], params[:remember_me])
  if user
    redirect_back_or_to user, :notice => "Logged in!"
  else
    @error_message = "Invalid Username or Password."
    render 'new'
  end
end
```
Create a file with name `show.html.erb` in `app/views/users/` folder and copy the following code snippet into that.

```rhtml
<h2>Welcome to Expense Report Application</h2>

<p>
  Hello <%= current_user.username %>,<br/>
  you are going to see a nice sample Expense Report application in some time.
</p>
```

We still need a way for the user to be able to log out. So we’ll also add a destroy action to the controller. `logout`, another method offered by Sorcery, is all we need to call to log the user out. Once that is done, we've logged a user out, and redirect them back to the home page. Add the following method in `app/controllers/sessions_controller.rb` file to carry that out.

```ruby
def destroy
  logout
  redirect_to root_url, :notice => "Logged out!"
end
```

## **Step 7:** Generating Routes

Next go to routes file `config/routes.rb` and wire everything up, replacing the default generated actions with these routes:

```ruby
RailsExpenseReportPostgres::Application.routes.draw do
  get "logout" => "sessions#destroy", :as => "logout"
  get "login" => "sessions#new", :as => "login"
  get "signup" => "users#new", :as => "signup"
  resources :users
  resources :sessions
  
  root :to => "sessions#new"
end
```

We now have various named routes and a couple of resources to handle all of the functionality related to authentication. The following command will list all the available routes in the application.

```bash
$ bundle exec rake routes
```

## **Step 8:** Adding Layout

`current_user` is a method available by `Sorcery` gem. This tells us whether we have a logged in user or not. If there is, then display their `username` along with a `Log out` link. If not display the links to `sign up or log in`. Copy the following code snippet into `app/views/layouts/application.html.erb` file

```rhtml
<!DOCTYPE HTML>
<html>
  <head>
    <title>RailsExpenseReportPostgres</title>
    <%= stylesheet_link_tag "application", :media => "all" %>
    <%= javascript_include_tag "application" %>
    <%= csrf_meta_tags %>
  </head>
  <body>
    <div id="contentWrapper">
      <div class="blue-b">
        <%= render :partial => "layouts/header_nav" %>
      </div>
      <div class="mc-a">
        <%= yield %>
      </div>
    </div>
  </body>
</html>
```

Code snippet given below is for the partial file `_header_nav.html.erb` which is being rendered in `application.html.erb` layout.

```rhtml
<div class="inside-nav">
  <div class="logo">
    <img src="<%= asset_path 'logo.png' %>">
  </div>

  <ul class="nav-a">
    <% if current_user %>
      <li><a href="javascript:void(0)">Hi <%= current_user.username %></a></li>
      <li>|</li>
      <li><%= link_to "Logout", logout_path %></li>
    <% else %>
      <li>
        <%= link_to "Login", login_path, :class => params[:controller].eql?('sessions') ? 'selected' : '' %>
      </li>
      <li>|</li>
      <li>
        <%= link_to "Signup", signup_path, :class => params[:controller].eql?('users') ? 'selected' : '' %>
      </li>
    <% end %>
  </ul> 
</div>
```

##Setup: Exercise2-Complete

If you want to test the application from the downloaded source code [Exercise2-Complete.zip](/rails-code/expense-report-postgres/Exercise2-Complete.zip), please setup the application as described [here](/frameworks/ruby/rails-tutorial/psql-completers-guide.html) before continue with *check point*.

##Check Point

We’re ready now to test the site out.

##Run the App Locally

Execute the following command to run the app

```bash
$ rails s
```

The server will run on port 3000. Open the browser and load the url `localhost:3000`.

![Image for Login page](/images/rails-tutorial/login-page.png)

Click on `Signup` link, then you will see a form to sign-up as shown below.

![Image for Signup page](/images/rails-tutorial/signup-page.png)

![Image for Signup page with error messages](/images/rails-tutorial/signup-with-errors.png)

Below is after successful Login

![Image after successful login](/images/rails-tutorial/user-show-page.png)

<a class="button-plain" style="padding: 3px 15px;" href="/frameworks/ruby/rails-tutorial/rails-new-template-with-postgres.html">Prev</a>   <a class="button-plain" style="padding: 3px 15px; float: right;" href="/frameworks/ruby/rails-tutorial/rails-expense-user-flow.html">Next</a>
