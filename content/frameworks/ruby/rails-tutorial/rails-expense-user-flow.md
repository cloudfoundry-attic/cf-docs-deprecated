---
title: Expense Reporting Application with Rails
description: Adding Expenses to Application
tags:
    - ruby
    - rails
    - expense_user_flow
---

##Introduction

This chapter is continuation for the [chapter-2](/frameworks/ruby/rails-tutorial/rails-user-login.html), and this section explains the process how to add the functionality to create, edit and delete an expense by a user.

##Prerequisites

+ You should complete the [chapter-2](/frameworks/ruby/rails-tutorial/rails-user-login.html), which lets you to add the authentication for our Expense reporting application.
+ [ImageMagick](http://www.imagemagick.org/script/install-source.php) to be installed on your system to upload attachments.

##Adding Expenses to Application

We are ready with the user login and sign-up pages in [chapter-2](/frameworks/ruby/rails-tutorial/rails-user-login.html) which are needed for checking the user authentication. 

##Subtopics

+ [Handling User Roles](#handling-user-roles)
+ [Adding Functionality for Expense Creation](#adding-functionality-for-expense-creation)

##Handling User Roles

As explained in [overview of the application](/frameworks/ruby/rails-tutorial/rails-new-template-with-postgres#expense-reporting-application-overview), the application contains two roles. So we have to handle the application based on the user role.

Basically here we have two roles, admin and normal user. Normal user should be able to create his own expenses and also able to edit or delete them, whereas an Admin can accept or reject the expenses. `CanCan` is an authorization library for Ruby on Rails which restricts what resources a given user is allowed to access.

Here in expense_reporting application we have only one action `expense-approval` which needs the access for the user based on his role, so simply we can go without any gem.

Here, i am going to take the roles in an array as a global constant, so that i can access that array of roles any where in the application. So create a file named `global_constants.rb` in `/config/initializers/` folder and keep the below line in that file.

```ruby
USER_ROLES = %w[admin user]
```

We need to restart the server if we change any configuration files. As we have added one new constants file,we have to restart the server. Now generate a migration file to add field `role` to users model. 

```bash
$ rails g migration add_role_to_users
    invoke  active_record
    create    db/migrate/20120917064703_add_role_to_users.rb
```

Now modify the generated migration file below to add the `role` field to users table.

```ruby
class AddRoleToUser < ActiveRecord::Migration
  def change
    add_column :users, :role, :string, :default => USER_ROLES[1]
  end
end
```

Here i am giving `normal user` as the default value for this field, so that each user who is signing up from the browser, will be a normal user. We can create an admin user by console. 

After making changes in the migration file, just run the following command. This will add the field in the table.

```bash
$ bundle exec rake db:migrate
```

Now write a method in user model `app/models/user.rb` which will tell whether the user is admin_user or not?

```ruby
def is_admin?
  role.eql? USER_ROLES[0]
end
```

The above method will return true if the user is admin else it will return false. Hense this method will be useful to handle the specific actions to check weather the user is having the admin access or not.

And make the field `role` as attribute_accesible in user model so that it wont throw any error for mass-assignment of user object. So, modify the `attr_accessible` line in `app/models/user.rb` file as shown below.

```ruby
attr_accessible :username, :password, :password_confirmation, :role
```

##Adding Functionality for Expense Creation

Now we will create the mvc for expense flow. First create a model for the ExpenseType

```bash
$ rails g model expense_type
    invoke  active_record
    create    db/migrate/20120917093435_create_expense_types.rb
    create    app/models/expense_type.rb
    invoke    test_unit
    create      test/unit/expense_type_test.rb
    create      test/fixtures/expense_types.yml
```

This will generate one migration file for ExpenseType model. Add one field called `name` in that table that refers to the type of an expense.

```ruby
class CreateExpenseTypes < ActiveRecord::Migration
  def change
    create_table :expense_types do |t|
      t.string :name
      t.timestamps
    end
  end
end
```

According to rails 3.1 modifications, whenever you create or update a model object in your controllers by using mass assignment you need to use attr_accessible in the models to protect the attributes. So modify the ExpenseType class as below. make the `name` attribute as attr_accessible, so that we can update this field in controller or any where.

```ruby
class ExpenseType < ActiveRecord::Base
  attr_accessible :name
  has_many :expenses
  validates :name, :presence => true, :uniqueness => true
end
```

As each expense_type has many expenses, as shown above, we need to specify the association in the class like `has_many :expenses`, which will be generated in the next step. and also given the validations like the `name` field should not be blank and should be unique.

Now generate a model for Expense, which is the master table for our application. 

```bash
$ rails g model expense
    invoke  active_record
    create    db/migrate/20120917095523_create_expenses.rb
    create    app/models/expense.rb
    invoke    test_unit
    create      test/unit/expense_test.rb
    create      test/fixtures/expenses.yml
```

An expense may have the fields like `description`, `amount`, `status` - for keeping track of the status of the expense whether it has been approved or rejected by the admin, and also having one attachment - either an image or a pdf for that particular transaction.

So for keeping track of the status of an expense, just add a global constant in `/config/initializers/global_constants.rb` file

```ruby
EXPENSE_STATUS = %w[pending accepted rejected]
```

For adding attachments, use the gem [carrierwave](https://github.com/jnicklas/carrierwave). It requires imagemagick to be installed on our machines. And also we need to add one field `attachment` with type string for managing the attachment.

Add the following gem to your Gemfile and then do `bundle install`

```ruby
gem 'carrierwave'
```

After `bundle install`, we need to run the following command to generate the uploader file.

```bash
$ rails generate uploader Attachment
```

This will generate a file `app/uploaders/attachment_uploader.rb` which will be useful to modify the configurations for uploading the attachment.

Finally, the migration file for expense creation will be like as follows.

```ruby
class CreateExpenses < ActiveRecord::Migration
  def change
    create_table :expenses do |t|
      t.string :description
      t.date :date
      t.float :amount
      t.string :status, :default => EXPENSE_STATUS[0]
      t.string :attachment

      t.references :expense_type
      t.references :user
      t.timestamps
    end
  end
end
```

Now run the command for creating the tables and fields.

```bash
$ bundle exec rake db:migrate
```

Modify the expense model file `app/models/expense.rb` making `attr_accessible` and also add the associations accordingly.

```ruby
class Expense < ActiveRecord::Base

  attr_accessible :description, :date, :amount, :status,
                  :attachment, :expense_type_id, :user_id
  
  mount_uploader :attachment, AttachmentUploader # for using the attachment_uploader.rb file for uploading the attachments
  
  belongs_to :user
  belongs_to :expense_type

  validates :description, :expense_type_id, :status, :user_id, :presence => true
  validates :amount, :numericality => {:message => "is not valid"}

end
```

Now its time to create the controller and view part for the expense creation. Expense will be created by a normal user only.

```bash
$ rails g controller expenses
```

Add the expense resources to routing file in `/config/routs.rb`. This will create all the basic routes.

```ruby
resources :expenses
```

The basic code for expense crud operations `new`, `create`, `edit`, `update`, `index` and `destroy` methods typically look like :

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
    @expense = Expense.find(params[:id])
  end

  def update
    @expense = Expense.find(params[:id])
    if @expense.update_attributes(params[:expense])
      flash[:notice] = "Expense updated successfully"
      redirect_to expenses_path
    else
      render "edit"
    end
  end

  def destroy
    expense = Expense.find(params[:id])
    expense.destroy
    redirect_to :back
  end
end
```

Here `current_user` and `require_login` methods are helper methods available because of sorcery gem, and `current_user.expenses.build` will create a new expense object for the current logged_in user. And in create method, if the object is created successfully, then we are redirecting the user to expense index page, else rendering the form with errors.

Now we have to create the view files needed for creating the expense.

in `app/views/expenses/new.html.erb`

```rhtml
<h2>Creating new Expense</h2>
<%= render :partial => "expenses/form" %>
```

In `app/views/expense/_form.html.erb`, we need to keep the form fields needed for creating an expense along with displaying errors, if any.

```rhtml
<%= form_for @expense, :html => {:multipart => true} do |f| %>
  <% if @expense.errors.any? %>
    <div class="error_messages">
      <ul>
        <% for message in @expense.errors.full_messages %>
          <li><%= message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <div>
    <%= f.label :description %>
    <%= f.text_area :description, :placeholder => "description" %>
  </div>

  <div>
    <%= f.label :amount %>
    <%= f.text_field :amount, :placeholder => "Amount like 210.50" %>
  </div>

  <div>
    <%= f.label :expense_type_id %>
    <%= collection_select(:expense, :expense_type_id, ExpenseType.all, :id, :name, options ={:prompt => "Select Expense Type"} ) %>
  </div>
  
  <div>
    <%= f.label :attachment %>
    <%= f.file_field :attachment %>
  </div>
  
  <div>
    <%= f.label :date, "Generated on :" %>
    <input type="text" name="expense[date]" placeholder="DD/MM/YY", id="date-field" />
  </div>
  <div>
    <%= f.submit "submit" %>
  </div>
<% end %>

<div><%= link_to "Back", expenses_path %></div>
```

Here for date field, we can use any datepickers. For example jquery-datepicker, a nice plugin for [datepicker](http://jqueryui.com/demos/datepicker/). Add the necessary files to the application for using this plugin. Go to [here](http://jqueryui.com/download) and first deselect all and then select only `datepicker` in `widgets` section, download them and then add the images to the `app/assets/images/` folder and css file to `app/assets/styles/` folder.

I am initializing the datepicker on the input field using the following code in `application.js` file

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

Add the content for `/app/views/expenses/edit.html.erb` file as follows

```rhtml
<h2>Edit Expense</h2>
<%= render :partial => "expenses/form" %>
```

The`/app/views/expenses/index.html.erb` file looks like the following.

```rhtml
<table>
  <thead>
    <tr>
      <td>Date</td>
      <td>Type</td>
      <td>Description</td>
      <td>Status</td>
      <td>Actions</td>
    </tr>
  </thead>
  <tbody>
    <%= render @expenses %>
  </tbody>
</table>

<% unless current_user.is_admin? %>
  <div>
    <%= link_to "Add new Expense", new_expense_path %>
  </div>
<% end %>
```

and `app/views/expenses/_expense.html.erb` will be

```rhtml
<tr>
  <td><%= I18n.localize(expense.date, :format => "%d-%I-%Y") if expense.date %></td>
  <td><%= expense.expense_type.name %></td>
  <td><%= expense.description %></td>
  <td><%= expense.status %></td>
  <td>
    <% unless current_user.is_admin? %>
      <%= link_to "Edit", edit_expense_path(expense) %>
      <%= link_to "Delete", expense_path(expense), :confirm => "Are you sure want to delete?", :method => :delete %>
    <% end %>
  </td>
</tr>
```

Previously we thought of redirecting the user to index page of expenses after successful login. Thus, we need to modify the `redirect_to` path in sessions_controller's `app/controllers/sessions_controller.rb` create method.

```ruby
redirect_back_or_to expenses_path(), :notice => "Logged in!"
```

Now I want to populate some data using `db/seeds.rb` file. So modify that file as below.

```ruby
puts "creating default user and admin"
User.create!(username: "user", password: "123123", password_confirmation: "123123")
User.create!(username: "admin", password: "123123", password_confirmation: "123123", role: USER_ROLES[0])

puts "creating expense types"
ExpenseType.create!(name: "Travel")
ExpenseType.create!(name: "Education")
ExpenseType.create!(name: "Health")
```

After modifying the seeds files, we need to populate that records.For that, run the following command in the terminal

```bash
$ bundle exec rake db:seed
```

Now open browser and login as user. You will be redirected to the expenses index page of that user.


[<= Prev](/frameworks/ruby/rails-tutorial/rails-user-login.html)  <span style="float: right;">[Next =>](/frameworks/ruby/rails-tutorial/rails-expense-admin-flow.html)</span>
