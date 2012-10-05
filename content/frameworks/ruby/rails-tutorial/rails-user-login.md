---
title: Expense Reporting Application with Rails
description: Adding User Authentication to Application
tags:
    - ruby
    - rails
    - user_login
---

##Introduction

This chapter is continuation of the [chapter-1](/frameworks/ruby/rails-tutorial/rails-new-template-with-postgres.html), and this section explains how to add the user authentication to Expense Reporting Application, so that a user can access only the authorized content in the application.

##Prerequisites

+ You should complete the chapter-1, which lets you to create a basic template and also configuring the database etc..
+ You should know how to use a gem in rails application

##Adding User Authentication to the application

Providing authentication is a basic need for any application. There are multiple ways for providing that functionality and there are lot of gems available in the [github](https://github.com/). Devise is one of the best gem to use for the authentication. List of [gems that provides authentication](https://www.ruby-toolbox.com/categories/rails_authentication).

Expense Reporting is an application with basic requirements. So, going with a simple authentication gem **sorcery** provides common authentication needs such as signing in/out, activating by email and resetting password. Explore the complete features for this gem here [Read Me doc](https://github.com/NoamB/sorcery) and [Wiki for Sorcery](https://github.com/NoamB/sorcery/wiki).

Add the gem to your gem list in Gemfile. This can be done by adding the below line to your `Gemfile`.

```ruby
gem 'sorcery'
```

Install the gem by running the following command.

```bash
$ bundle install
```

Now we are ready to use the sorcery gem in our application. First we need to generate the files required to configure the gem. Run the following command to generate those.

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
      create  db/migrate/20120913091959_sorcery_core.rb
```

As shown above, this will generate the initializer file for configuring the settings, user model class without migration, unit test stubs, and sorcery_core migration file for adding the user objects.

To enable Sorcery authentication in the User model, Sorcery will generate the User model by default with the following line. This adds a number of methods to the User model to handle authentication.

```ruby
class User < ActiveRecord::Base
  authenticates_with_sorcery!
end
```

Basically, Sorcery is capable of giving the authentication based on either username or email. So it will generate a migration file with both fields email and usernames. In this application, we are going to give the authentication based on username, so we can comment out or remove the field email from the sorcery_core migration file here in this case it is `db/migrate/20120913091959_sorcery_core.rb`. `20120913091959` this is based on the time at which the file is generated.

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

Sorcery gem will generate all these fields by default. And if we want to change the names of the fields, we need to tell the sorcery about our modifications. Sorcery will provide a lot of options to customize. You will see a list of all options in `config/initializers/sorcery.rb` file.

As in this application, we are not going to have the things like remember me or, reset password. If we want to include them, we need to generate the migration files and then need to add those sub modules in sorcery configuration file. Follow this for [Remember me](https://github.com/NoamB/sorcery/wiki/Remember-Me) and [Reset Password](https://github.com/NoamB/sorcery/wiki/Reset-password)

**Note:** If you want to add any sub modules for example `Remember_me`, in the wiki pages, they started with `rails g sorcery:install remember_me`, which will generate the configurtion file for sorcery reset all the modifications if any. As already we generated the configuration files, we have to generate only the required migration files. This can be done by executing `rails g sorcery:install remember_me --migrations`. This will generate only the migration files. Then follow the next steps explained in the wiki.

Once you have modified the sorcery configuration file and migration files accordingly, setup the database by the following command.

```ruby
bundle exec rake db:migrate
```

Now, its our wish to give any add validations or protect any attributes for User model attributes. Our User model doesn’t have password or password_confirmation attributes but we’ll be making accessible methods inside the User model to handle these.

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

Now that we have the User model set up, we’ll generate some controllers to go with it. First we’ll create a UsersController to handle the sign-up process.

```bash
$ rails g controller users new
```

Next we’ll write the code for the new and create actions in the UsersController. The following code should go into the file /app/controllers/users_controller.rb

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

This is fairly standard controller code. In the new action we create a new User while in the create action we create a new User based on the parameters that are passed in (these will come from a form) . If that new User is valid then we redirect to the home page (which we don’t have yet), otherwise we render the new action template again. Here we have writter `before_filter :require_login`, this method is provided by the sorcery gem, this will allow user to particular actions only if he is logged 

We’ll write that new template now. It will have a form that has email, password and password_confirmation fields along with some code for showing any validation errors. Hence the following code should go into /app/views/users/new.html.erb

```rhtml
<h1>Sign Up</h1>

<%= form_for @user do |f| %>
  <% if @user.errors.any? %>
    <div class="error_messages">
      <h2>Form is invalid</h2>
      <ul>
        <% for message in @user.errors.full_messages %>
          <li><%= message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>
  <div>
    <%= f.label :username %>
    <%= f.text_field :username %>
  </div>
  <div>
    <%= f.label :password %>
    <%= f.password_field :password %>
  </div>
  <div>
    <%= f.label :password_confirmation %>
    <%= f.password_field :password_confirmation %>
  </div>
  <div>
    <%= f.submit %>
  </div>
<% end %>
```

We’re half-way there now. Users can sign up but they can’t yet sign in. We’ll fix that now, creating a new controller called sessions that will handle to login form.

```bash
$ rails g controller sessions new
```

Things get more interesting in the SessionsController. We have a new action here, but we don’t need to add any code to it so we’ll move on to the template. Inside the new view file that was generated we’ll create the form for signing in.

```rhtml
<h1>Log in</h1>

<%= form_tag sessions_path do %>
  <div class="field">
    <%= label_tag :username %>
    <%= text_field_tag :username, params[:username] %>
  </div>
  <div class="field">
    <%= label_tag :password %>
    <%= password_field_tag :password %>
  </div>
  <div class="actions"><%= submit_tag "Log in" %></div>
<% end %>
```

We use form_tag rather than form_for here as form_for implies that there’s a resource behind the form. We don’t have a Session model so this is not the case here. The form POSTs to sessions_path, which will be the SessionController’s create action. The form has two fields, one for the username and one for the password.

We’ll need to write a create action to process the login form. Sorcery provides a method called login that can take three parameters: a username or email address, a password that was entered and the value of the remember_me checkbox. This method will perform the authentication and return a User if a matching one is found. We can check for this and redirect back to the home page if a user is found. If not we’ll display a flash message and show the login form again. So the following code will goes to `/app/controllers/sessions_controller.rb`

```ruby
class SessionsController < ApplicationController
  def new
  end

  def create
    user = login(params[:username], params[:password], params[:remember_me])
    if user
      redirect_back_or_to root_url, :notice => "Logged in!"
    else
      flash.now.alert = "Username or password was invalid."
      render 'new'
    end
  end
end
```

Instead of using redirect_to to redirect back to the home page when a user is found we will use a method that Sorcery gives us called redirect_back_or_to. This behaves similar to redirect_to but if a URL is stored by Sorcery it will redirect back to that URL rather than to the one specified in the code. This is useful because it means that if we require a user to log in when they visit a certain page, Sorcery will take them to the login page and then return them back to the page they tried to visit once they had successfully logged in.

On the user's successful signin, he should be redirect_to his expenses index page. As for now, we have not implemented the expenses flow. So, we  temporarily make him redirect to show action in users controller.Modify the create method in sessions_controller.rb file accordingly.

```ruby
def create
  user = login(params[:username], params[:password], params[:remember_me])
  if user
    redirect_back_or_to user, :notice => "Logged in!"
  else
    flash.now.alert = "Username or password was invalid."
    render 'new'
  end
end
```
Now we shall create a show page in apps/views/users 

```rhtml
<h2>Welcome to Expense Report Application</h2>

<p>Hello <%= current_user.username %> are going to see a nice sample Expense Report application in some time...</p>
```

We still need a way for the user to be able to log out so we’ll also add a destroy action to the controller. Sorcery provides a method called logout and that’s all we need to call to log the user out. Once we’ve logged a user out we’ll redirect them back to the home page. So add the following method in `sessions_controller.rb` file

```ruby
def destroy
  logout
  redirect_to root_url, :notice => "Logged out!"
end
```

Next we’ll go into our routes file and wire everything up, replacing the default generated actions with these routes:

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

We now have various named routes and a couple of resources to handle all of the functionality related to authentication. If you want to know what are all the routes just check in the terminal by using the below commands.

```bash
$ bundle exec rake routes
```

Now that we have these new pages we’ll need some links so that users can access them. We’ll add them to the layout page so that they are visible on every page of the application. We have a `current_user` method available to us so that we can check to see if we have a logged-in user. If so, we’ll show their `username` along with a “Log out” link. If there’s no current user we’ll show links to pages that will let them either sign up or log in.
Modify your layout as follows. Keep the below code in your /app/views/layouts/application.html.erb

```rhtml
<!DOCTYPE html>
<html>
<head>
  <title>Expense Reporting</title>
  <%= stylesheet_link_tag    "application" %>
  <%= javascript_include_tag "application" %>
  <%= csrf_meta_tags %>
</head>
<body>
  <div id="container">
    <div class="user_nav">
      <% if current_user %>
        Logged in as <%= current_user.email %>.
        <%= link_to "Log out", logout_path %>
      <% else %>
        <%= link_to "Sign up", signup_path %> or
        <%= link_to "Log in", login_path  %>.
      <% end %>
    </div>
    <% flash.each do |name, msg| %>
      <%= content_tag :div, msg, :id => "flash_#{name}" %>
    <% end %>
    <%= yield %>
  </div>
</body>
</html>
```

We’re ready now. to test the site out. Start the server first to test the app, and you can start by executing the following command.

```bash
$ rails s
```

The server will run on port 3000. So open browser and go to the url `localhost:3000`. If we go to the home page we’ll now see the “Sign up” and “Log in” links.

![Image for Login page](/images/rails-login-page.png)


[<= Prev](/frameworks/ruby/rails-tutorial/rails-new-template-with-postgres.html)   <span style="float: right;">[Next =>](/frameworks/ruby/rails-tutorial/rails-expense-user-flow.html)</span>
