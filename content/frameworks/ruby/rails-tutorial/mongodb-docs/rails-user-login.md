---
title: Expense Reporting Application with Rails
description: Adding User Authentication to Application
tags:
    - ruby
    - rails
    - user_login
    - mongodb
---

##Introduction

This chapter is a continuation of [chapter 1](/frameworks/ruby/rails-tutorial/mongodb-docs/rails-new-template-with-mongodb.html), and this section explains adding user authentication to the Expense Reporting Application, so that a user can access only the authorized content in the application.

##Prerequisites

+ Should have completed the [chapter 1](/frameworks/ruby/rails-tutorial/mongodb-docs/rails-new-template-with-mongodb.html), that explains creating a basic template, configuring the database etc...
+ Should know how to use a gem in a Rails application

##Adding User Authentication to the application

Authentication is a basic need for any application. There are multiple ways for providing that functionality and there are a lot of gems available on [github](https://github.com/). `Devise` is one of the best gem to use for the authentication. This is the list of [gems that provides authentication](https://www.ruby-toolbox.com/categories/rails_authentication).

Expense Reporting is an application with basic requirements. A simple authentication gem **sorcery** provides common authentication needs such as signing in/out, activating by email and resetting password. Explore the complete features of this gem here [Read Me doc](https://github.com/NoamB/sorcery) and [Wiki for Sorcery](https://github.com/NoamB/sorcery/wiki).

Add the gem to your gem list in Gemfile. This can be done by adding the below line to your `Gemfile`. You can find the file  `/Gemfile` in the root folder of the application

```ruby
gem 'sorcery'
```

Install the gem by running the following command.

```bash
$ bundle install
```

The sorcery gem can now be used in our application. Run the following command to generate the files required to configure the gem.

```bash
$ rails generate sorcery:install
      create  config/initializers/sorcery.rb
    generate  model User --skip-migration
      invoke  mongoid
      create    app/models/user.rb
      invoke    test_unit
      create      test/unit/user_test.rb
      create      test/fixtures/users.yml
      insert  app/models/user.rb
```

As demonstrated, this will generate the initializer file for configuring the settings, user model class, unit test stubs.

As we are working with mongodb, there is no such things like migration files, hence Sorcery will not generate any fields for the user mode. we have to add explicitly what ever we needed. And to enable Sorcery authentication in the User model, add the line `authenticates_with_sorcery!` in the User model, this adds a number of methods to the User model to handle authentication. So modify the user model as mentioned below.

```ruby
class User
  include Mongoid::Document
  include Mongoid::Timestamps

  authenticates_with_sorcery!
  
  field :username, type: String
  field :crypted_password, type: String
  field :salt, type: String

  attr_accessor :password, :password_confirmation

  validates_confirmation_of :password
  validates_presence_of :password, :on => :create
  validates_presence_of :username
  validates_uniqueness_of :username

end

```

As described in above code snippet, we also added validations for User model attributes. Our User model doesn’t have password or password_confirmation attributes but we used accessible methods inside the User model to handle them.

Sorcery offers a lot of options for customization. You will see a list of all options in `config/initializers/sorcery.rb` file.

Now that we have the User model set up, we’ll generate some controllers to go with it. First we’ll create a UsersController to handle the sign-up process.

```bash
$ rails g controller users new
```

Next we’ll write the code for the new and create actions in the UsersController. The following code should go into the file `/app/controllers/users_controller.rb`

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

This is fairly standard Controller code. In the `new` action we create a new User object and in the `create` action we create a new User based on the parameters that are passed in (these will come from a form) . If the New User is valid, we shall redirect him to the home page (which we don’t have yet), otherwise we render the new action template with error messages. Here we have writter `before_filter :require_login`, this method `require_login` is a helper method provided by the sorcery gem, this will allow user to perform particular actions only if they are logged in.

Copy the following snippet of code into `app/views/users/new.html.erb` file. This code gives a form consisting of `email, password and password_confirmation` fields along with some code for displaying validation errors if any.

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

Users can Sign up but can’t Signin yet. We will Fix that by creating a new controller called `sessions_controller` that will handle to login and logout.

```bash
$ rails g controller sessions new
```

We will use the default implementation of action `new` in `app/cntrollers/sessions_controller.rb` so we’ll move on to the view part for this. In `app/views/sessions/new.html.erb` file we shall create the form for signing in. Hence the following snippet of code should go into `/app/views/sessions/new.html.erb`

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

We use `form_tag` rather than `form_for` here as form_for implies that there’s a resource behind the form. Since we don’t have a Session model, this is not the case here. The form POSTs to sessions_path, which will be the SessionController’s `create` action. The form has two fields, one each for username and password.

We’ll need to write a `create` action to process the login form. Sorcery's Login method can take three parameters: a username or email address, a password that was entered and the value of the remember_me checkbox. This method authenticates and return a User if a matching one is found. We can check for this and redirect back to the home page if a user is found. If not we’ll display a flash message and show the login form again. So the following code will goes to `/app/controllers/sessions_controller.rb`

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

Instead of using redirect_to to redirect back to the home page when a user is found we will use a method that Sorcery gives us called redirect_back_or_to. This behaves similar to redirect_to but if a URL is stored by Sorcery it will redirect back to that URL rather than to the one specified in the code. This is useful because it means that if we require a user to log in when they visit a certain page, Sorcery will take them to the login page and then return them back to the page they tried to visit once they had successfully logged in.

Once the signin successful, the User should be redirect_to his expenses index page. We have not implemented the expenses flow. So for time being, let us redirect the User to show action in users controller. Modify the `create` method in sessions_controller.rb file accordingly.

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

We will now create a show page here `apps/views/users/show.html.erb`

```rhtml
<h2>Welcome to Expense Reporting Application</h2>

<p>Hello <%= current_user.username %>, you are going to see a nice sample Expense Reporting application in some time...</p>
```

We still need a way for the user to be able to log out so we’ll also add a destroy action to the controller. Logout, another method offered by Sorcery, is all we need to call to log the user out. Once that is done, we've logged a user out, and direct them back to the home page. Add the following method in `sessions_controller.rb` file to carry that out.

```ruby
def destroy
  logout
  redirect_to root_url, :notice => "Logged out!"
end
```

Next we’ll go into our routes file `/config/routes.rb` and wire everything up, replacing the default generated actions with these routes:

```ruby
ExpenseReporting::Application.routes.draw do
  get "logout" => "sessions#destroy", :as => "logout"
  get "login" => "sessions#new", :as => "login"
  get "signup" => "users#new", :as => "signup"
  resources :users
  resources :sessions
  
  root :to => "sessions#new"
end
```

We now have various named routes and a couple of resources to handle all of the functionality related to authentication. If you want to know what are all the routes, just check in the terminal by using the below commands.

```bash
$ bundle exec rake routes
```

We have a `current_user` method available by `sorcery` gem, using this we can check whether we have a logged-in user or not. If so, we’ll show their `UserName` along with a `Log out` link. If there’s no current user we’ll show links to pages that will let them either `sign up or log in`.
. Copy the following code snippet into `app/views/layouts/application.html.erb` file


```rhtml
<!DOCTYPE HTML>
<html>
  <head>
    <title>ExpenseReporting</title>
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

Code snippet given below is for the partial `_header_nav.html.erb` which is rendering above in the `application.html.erb` layout

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

##Check Point 

We’re ready now. to test the site out. Start the server first to test the app, and you can start by executing the following command.

```bash
$ rails s
```

The server will run on port 3000. So open your browser and go to the url `localhost:3000`. If we go to the home page we’ll now see the “Log in” and “Sign up” links and also a form to Login.

![Image for Login page](/images/rails-tutorial/login-page.png)

Click on `Signup` link, then you will see a form to sign-up as shown below.

![Image for Signup page](/images/rails-tutorial/signup-page.png)

![Image for Signup page with error messages](/images/rails-tutorial/signup-with-errors.png)

Below is after successful Login

![Image after successful login](/images/rails-tutorial/user-show-page.png)

<a class="button-plain" style="padding: 3px 15px;" href="/frameworks/ruby/rails-tutorial/mongodb-docs/rails-new-template-with-mongodb.html">Prev</a>   <a class="button-plain" style="padding: 3px 15px; float: right;" href="/frameworks/ruby/rails-tutorial/mongodb-docs/rails-expense-user-flow.html">Next</a>
