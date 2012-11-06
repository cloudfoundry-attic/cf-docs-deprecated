---
title: Expense Reporting Application with Rails
description: Adding Expenses to the Application
tags:
    - ruby
    - rails
    - expense_user_flow
---

##Introduction

This chapter is a continuation of [chapter-2](/frameworks/ruby/rails-tutorial/mongodb-docs/rails-user-login.html). This section will teach you how to add the functionality for users to create, edit and delete expenses.

##Prerequisites

+ Should have completed [chapter 2](/frameworks/ruby/rails-tutorial/mongodb-docs/rails-user-login.html) - that explains the authentication process for our Expense reporting application.
+ [ImageMagick](http://www.imagemagick.org/script/install-source.php) must be installed on your system to upload attachments (It is already available on Mac but needs to be installed for Ubuntu or Windows).

##Adding Expenses to Application

The user login and sign-up pages needed for authenticating the users have been created in [chapter-2](/frameworks/ruby/rails-tutorial/mongodb-docs/rails-user-login.html).

##Subtopics

+ [Handling User Roles](#handling-user-roles)
+ [Adding Functionality for Expense Creation](#adding-functionality-for-expense-creation)

##Handling User Roles

As explained in [the overview](/frameworks/ruby/rails-tutorial/mongodb-docs/rails-new-template-with-postgres#expense-reporting-application-overview), the application operates based on the user's role.

There are two roles in the application, admin and normal user. Normal Users should be able to create their own expenses and be able to edit or delete them. An Admin can accept or reject expenses. `CanCan` is an authorization library for Ruby on Rails which restricts the resources a given user is allowed to access.

In the expense_reporting application we have only one action `expense-approval` that needs access for the user based on his role, so we don't need to use a gem.

## **Step 1:** Adding User Role

Consider the roles in an array as a global constant, so that the array of roles can be accessed from any where in the application. Create a file named `global_constants.rb` in `config/initializers/` folder and enter the below line in it.

```ruby
USER_ROLES = %w[admin user]
```

The server needs to be restarted if any configuration files are changed. As we have added a new Constants file, we have to restart the server. 

Add a field called `role` in User model, with default value as `user`. Modify `app/models/user.rb` file by adding the below line

```ruby
field :role, type: String, default: USER_ROLES[1]
```

`user` has been given as the default value for this field, so every user who signs up from the browser will be a normal user. An admin user can be created by console.

Implement a method in user model `app/models/user.rb` to tell if the user role is `admin` or `user`

```ruby
def is_admin?
  role.eql? USER_ROLES[0]
end
```

##Adding Functionality for Expense Creation

## **Step 2:** Adding ExpenseType and Expense Models

Let us create the mvc for expense flow. First create a model for the ExpenseType

```bash
$ rails g model expense_type
      invoke  mongoid
      create    app/models/expense_type.rb
      invoke    test_unit
      create      test/unit/expense_type_test.rb
      create      test/fixtures/expense_types.yml
```

Add the `name` field in the ExpenseType model and also add validation for this as mentioned below. Hence the code snippet in `app/models/expense_type.rb` file looks like below.

```ruby
class ExpenseType  
  include Mongoid::Document
  include Mongoid::Timestamps
  
  field :name, type: String

  has_many :expenses
  validates :name, :presence => true, :uniqueness => true
end
```

In the above code snippet, `has_many :expenses` refers to the association between `ExpenseType` model and `Expense` model (which will be generated in the next step). As each `expense_type` (class name ExpenseType) has many expenses (class name Expense), there is a 1:N association between expense_type and expense. And validations added for `name` field to test the presence and uniqueness of that attribute.

Generate a model for Expense, which is the master table for our application.

```bash
$ rails g model expense
      invoke  mongoid
      create    app/models/expense.rb
      invoke    test_unit
      create      test/unit/expense_test.rb
      create      test/fixtures/expenses.yml
```

An `Expense` will have fields like `description`, `amount` and `status` for keeping track of the status of an expense. It will also have an ttachment - either an image or a pdf.

To keep track of the status of an expense, add a global constant in `config/initializers/global_constants.rb` file

```ruby
EXPENSE_STATUS = %w[pending accepted rejected]
```

## **Step 3:** Adding Uploader for Attachments

To add attachments, use the [carrierwave-mongoid](https://github.com/jnicklas/carrierwave-mongoid) gem. The gem requires ImageMagick to be installed on our machines. A field with name `attachment` of type `String` should be added to the Expense model to handle the attachment uploading.

Add the following gem to your Gemfile

```ruby
gem 'carrierwave-mongoid'
```

Now install the gem by running the following command

```bash
$ bundle install
```

After `bundle install`, run the following command to generate the uploader file.

```bash
$ rails g uploader Attachment
```

This will generate a file `app/uploaders/attachment_uploader.rb` which will be used to modify the configurations for uploading the attachment.

Finally, add the required fields, validations and associations for Expense model in `app/models/expense.rb` file as given below

```ruby
class Expense
  include Mongoid::Document
  include Mongoid::Timestamps
  
  field :description, type: String
  field :amount, type: Float
  field :status, type: String, default: EXPENSE_STATUS[0]
  field :date, type: Date

  mount_uploader :attachment, AttachmentUploader
  
  belongs_to :user
  belongs_to :expense_type

  validates :description, :expense_type_id, :status, :user_id, :presence => true
  validates :amount, :numericality => {:message => "is not valid"}

  belongs_to :expense_type
  belongs_to :user

  scope :pending, where(:status => EXPENSE_STATUS[0])
end
```

As each user (class name User) has many expenses (class name Expense), there is a 1:N association between user and expense. Add the following line to User model in `app/models/user.rb` file to represent this association.

```ruby
has_many :expenses
```

## **Step 4:** Adding Expense Controller

Now its time to create the controller and view part for the expense creation.

```bash
$ rails g controller expenses
```

Add the expense resources to routing file in `config/routs.rb`. This will create the basic routes. Learn more about [routing](http://guides.rubyonrails.org/routing.html#singular-resources).

```ruby
resources :expenses
```

The basic code for expense crud operations `new`, `create`, `edit`, `update`, `index` and `destroy` methods typically look like below

```ruby
class ExpensesController < ApplicationController
  before_filter :require_login
  
  def new
    @expense = current_user.expenses.build
  end

  def create
    @expense = current_user.expenses.build(params[:expense])
    if @expense.save
      redirect_to expenses_path()
    else
      render "new"
    end
  end

  def index
    @expenses = current_user.expenses
  end

  def edit
    @expense = current_user.expenses.find(params[:id])
  end

  def update
    @expense = current_user.expenses.find(params[:id])
    if @expense.update_attributes(params[:expense])
      flash[:notice] = "Expense updated successfully"
      redirect_to expenses_path
    else
      render "edit"
    end
  end

  def destroy
    expense = current_user.expenses.find(params[:id])
    expense.destroy
    redirect_to :back
  end
end
```

Here `current_user` and `require_login` methods are helper methods available because of sorcery gem, and `current_user.expenses.build` will create a new expense object for the current logged_in user. In the `create` method, if the object is created successfully, the user is redirected to the expense index page or is rendered the form with errors if unsuccessful.

## **Step 5:** Adding View Part

Now let us create the view files needed for creating an expense.

in `app/views/expenses/new.html.erb`, copy the following line.

```rhtml
<%= render :partial => "expenses/form" %>
```

In `app/views/expense/_form.html.erb`, we need to have form fields needed for creating an expense and also be able to display errors, if any.

The code for `app/views/expense/_form.html.erb` is available in [this](/frameworks/ruby/rails-tutorial/code/chapter-3/view-files.html#content-for-expense-form) location.

For the date field, we can use any datepickers. jquery-datepicker, for example is a nice plugin for [datepicker](http://jqueryui.com/demos/datepicker/). We can initialize the datepicker on the input field by using the following code in `app/assets/javascripts/application.js` file

```js
//= require jquery
//= require jquery_ujs
//= require jquery-ui
//= require_tree .

$(document).ready(function(){
  $('#date-field').datepicker({
    dateFormat: 'dd/mm/yy'
  });
});
```

Create a file with name `edit.html.erb` in `app/views/expenses/` folder and copy the following line into it.

```rhtml
<%= render :partial => "expenses/form" %>
```

Create a file with name `index.html.erb` in `app/views/expenses/` folder and copy the following line into it.

```rhtml
<table class="table-a mtop80">
  <thead>
  <tr>
    <td class="td-a">
      <h5 class="label-3">Date</h5>
    </td>
    <td class="td-b">
      <h5 class="label-3">Type</h5>
    </td>
    <td  class="td-c">
      <h5 class="label-3">Description</h5>
    </td>
    <td  class="td-d">
      <h5 class="label-3">Status</h5>
    </td>
  </tr>
  </thead>
  <tbody>
    <%= render @expenses %>
  </tbody>
</table>
```

Create a partial file with name `_expense.html.erb` in `app/views/expenses/` folder and copy the code from [here](/frameworks/ruby/rails-tutorial/code/chapter-3/view-files.html#content-for-expense-object) and place in it.

## **Step 6:** Modifying the Layout

To navigate from index page to new expense page, add the navigation links in the header. Refer to the code snippet below on how this can be done.

Modify the `app/layouts/application.html.erb` as mentioned below

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
      <div class= "blue-b">
        <%= render :partial => "layouts/header_nav" %>
      </div>
      <% if current_user %>
        <%= render :partial => "layouts/actions_nav" %>
      <% end %>
      <div class= "mc-a">
        <%= yield %>
      </div>
    </div>
  </body>
</html>
```

Create a partial file with name `_actions_nav.html.erb` in `app/views/layouts/` folder and keep the below code snippet in this file. This file is being rendered in the `application.html.erb` layout.

```rhtml
<div id="nav">
  <ul>
    <li class="<%= params[:action].eql?('index') ? 'selected' : '' %>">
      <%= link_to "My Expenses", expenses_path %>
    </li>
    <li class="<%= ['index', 'approvals'].include?(params[:action]) ? '' : 'selected' %>">
      <%= link_to "New Expense", new_expense_path %>
    </li>
  </ul>
</div>
```

To redirect the user to Expenses index page after successful logging in, modify the `redirect_back_or_to` path in sessions_controller's `app/controllers/sessions_controller.rb` create method as shown below.

```ruby
redirect_back_or_to expenses_path(), :notice => "Logged in!"
```

## **Step 7:** Populating Data

`db/seeds.rb` file will be used to populate data. Modify the file like shown below.

```ruby
puts "creating default user and admin"
User.create!(username: "user", password: "123123", password_confirmation: "123123")
User.create!(username: "admin", password: "123123", password_confirmation: "123123", role: USER_ROLES[0])

puts "creating expense types"
ExpenseType.create!(name: "Travel")
ExpenseType.create!(name: "Education")
ExpenseType.create!(name: "Health")
```

After modifying the seeds files, run the following command to populate the data.

```bash
$ bundle exec rake db:seed
```

##Check Point

**Run the App Locally**

Execute the following command to run the app

```bash
$ rails s
```

Open browser and login with any of the user credentials as mentioned in step-7. The user will be redirected to his expenses index page. Then click on `New Expense` link. You will see as shown below

![Image for creating new Expense](/images/rails-tutorial/new-expense-creation.png)

Below Image is a screen with some expenses in index page

![Image for Exense index page after creating Expenses](/images/rails-tutorial/expense-index.png)

<a class="button-plain" style="padding: 3px 15px;" href="/frameworks/ruby/rails-tutorial/mongodb-docs/rails-user-login.html">Prev</a>  <a class="button-plain" style="padding: 3px 15px; float: right;" href="/frameworks/ruby/rails-tutorial/mongodb-docs/rails-expense-admin-flow.html">Next</a>
