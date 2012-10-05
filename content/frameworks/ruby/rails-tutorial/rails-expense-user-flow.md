---
title: Expense Reporting Application with Rails
description: Adding Expenses to the Application
tags:
    - ruby
    - rails
    - expense_user_flow
---

##Introduction

This chapter is a continuation of [chapter-2](/frameworks/ruby/rails-tutorial/rails-user-login.html). This section will teach you how to add the functionality for users to create, edit and delete expenses.

##Prerequisites

+ Should have completes [chapter 2](/frameworks/ruby/rails-tutorial/rails-user-login.html) - that explains the authentication process for our Expense reporting application.
+ [ImageMagick](http://www.imagemagick.org/script/install-source.php) must be installed on your system to upload attachments.

##Adding Expenses to Application

The user login and sign-up pages needed for authenticationg the users have been created in [chapter-2](/frameworks/ruby/rails-tutorial/rails-user-login.html).

##Subtopics

+ [Handling User Roles](#handling-user-roles)
+ [Adding Functionality for Expense Creation](#adding-functionality-for-expense-creation)

##Handling User Roles

As explained in [the overview](/frameworks/ruby/rails-tutorial/rails-new-template-with-postgres#expense-reporting-application-overview), the application operates based on the user's role.

We have two roles, admin and normal user. Normal Users should be able to create their own expenses and be able to edit or delete them. An Admin can accept or reject expenses. `CanCan` is an authorization library for Ruby on Rails which restricts the resources a given user is allowed to access.

In the expense_reporting application we have only one action `expense-approval` that needs access for the user based on his role, so we don't need to use a gem.

Let us take the roles in an array as a global constant, so that we can access that array of roles any where in the application. Create a file named `global_constants.rb` in `/config/initializers/` folder and enter the below line in it.

```ruby
USER_ROLES = %w[admin user]
```

The server needs to be restarted if any configuration files are changed. Since we have added a new Constants file,we have to restart the server. Generate a migration file to add the `role` field to Users model. 

```bash
$ rails g migration add_role_to_users
    invoke  active_record
    create    db/migrate/20121117064703_add_role_to_users.rb
```

Now modify the generated migration file below to add the `role` field to users table.

```ruby
class AddRoleToUser < ActiveRecord::Migration
  def change
    add_column :users, :role, :string, :default => USER_ROLES[1]
  end
end
```

`normal user` has been given as the default value for this field, so every user who signs up from the browser will be a normal user. An admin user can be created by console. 

After making changes in the migration file, just run the following command. This will add the field in the table.

```bash
$ bundle exec rake db:migrate
```

Now write a method in user model `app/models/user.rb` to tell if the user is admin_user or not

```ruby
def is_admin?
  role.eql? USER_ROLES[0]
end
```

The above method will return `true` if the user is admin and false if not. This method is useful in handling specific actions to check whether a user has admin access or not.

Make the `role` filed as attribute_accesible in the User model so that it doesn't throw an error for mass-assignment of user object. For that, modify the `attr_accessible` line in `app/models/user.rb` file as shown below.

```ruby
attr_accessible :username, :password, :password_confirmation, :role
```

##Adding Functionality for Expense Creation

Let us create the mvc for expense flow. First create a model for the ExpenseType

```bash
$ rails g model expense_type
    invoke  active_record
    create    db/migrate/20120917093435_create_expense_types.rb
    create    app/models/expense_type.rb
    invoke    test_unit
    create      test/unit/expense_type_test.rb
    create      test/fixtures/expense_types.yml
```

This will generate one migration file for ExpenseType model. Add one field called `name` in the table that will refer to the type of expense.

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

According to Rails 3.1 new features, whenever you create or update a model object in your controllers by using mass assignment, you need to use attr_accessible in the models to protect the attributes. So modify the ExpenseType class as demonstrated below. Make the `name` attribute as attr_accessible, so that we can update this field in controller or any where else.

```ruby
class ExpenseType < ActiveRecord::Base
  attr_accessible :name
  has_many :expenses
  validates :name, :presence => true, :uniqueness => true
end
```

As each expense_type has many expenses as shown above, we must specify the association in the class like `has_many :expenses`, which will be generated in the next step. Also, the validations like the `name` field should not be left blank and must be unique.

Let us next generate a model for Expense, which is the master table for our application. 

```bash
$ rails g model expense
    invoke  active_record
    create    db/migrate/20121117095523_create_expenses.rb
    create    app/models/expense.rb
    invoke    test_unit
    create      test/unit/expense_test.rb
    create      test/fixtures/expenses.yml
```

An expense may have fields like `description`, `amount`, `status` - for keeping track of the status of the expense that is whether it has been approved or rejected by the admin, it can also have one attachment - either an image or a pdf for that particular transaction.

To keep track of the status of an expense, add a global constant in `/config/initializers/global_constants.rb` file

```ruby
EXPENSE_STATUS = %w[pending accepted rejected]
```

To add attachments, use the [carrierwave](https://github.com/jnicklas/carrierwave) gem. The gem requires imagemagick to be installed on our machines. We also need to add an `attachment` field with type string for managing the attachment.

Add the following gem to your Gemfile and then do `bundle install`

```ruby
gem 'carrierwave'
```

After `bundle install`, run the following command to generate the uploader file.

```bash
$ rails generate uploader Attachment
```

This will generate a file `app/uploaders/attachment_uploader.rb` which will be used to modify the configurations for uploading the attachment.

Finally, the migration file for expense creation `db/migrate/20121117095523_create_expenses.rb` will be like follows.

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

Run the following command for creating the tables and fields.

```bash
$ bundle exec rake db:migrate
```

Modify the expense model file `app/models/expense.rb` add `attr_accessible` for accessibility and also add the associations accordingly.

```ruby
class Expense < ActiveRecord::Base

  attr_accessible :description, :date, :amount, :status,
                  :attachment, :expense_type_id, :user_id
  
  mount_uploader :attachment, AttachmentUploader # for using the attachment_uploader.rb file for uploading the attachments
  
  belongs_to :user
  belongs_to :expense_type

  validates :description, :expense_type_id, :status, :user_id, :presence => true
  validates :amount, :numericality => {:message => "is not valid"}

  scope :pending, where(:status => EXPENSE_STATUS[0])

end
```

Now its time to create the controller and view part for the expense creation.

```bash
$ rails g controller expenses
```

Add the expense resources to routing file in `/config/routs.rb`. This will create the basic routes.

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

Now let us create the view files needed for creating the expense.

in `app/views/expenses/new.html.erb`

```rhtml
<%= render :partial => "expenses/form" %>
```

In `app/views/expense/_form.html.erb`, we need to have form fields needed for creating an expense and also be able to display errors, if any.

[Here you can get the code for the view file](/frameworks/ruby/rails-tutorial/code/chapter-3/view-files.html#content-for-expense-form)


For the date field, we can use any datepickers. jquery-datepicker, for example is a nice plugin for [datepicker](http://jqueryui.com/demos/datepicker/). Add the necessary files to the application to usw this plugin. Go [here](http://jqueryui.com/download), deselect all and then select only `datepicker` in the `widgets` section. Download them and add the images to the `app/assets/images/` folder and css file to `app/assets/styles/` folder.

We can initialize the datepicker on the input field by using the following code in `application.js` file

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

Add the content for `/app/views/expenses/edit.html.erb` file as shown below

```rhtml
<%= render :partial => "expenses/form" %>
```

The`/app/views/expenses/index.html.erb` file looks like as given below

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

and `app/views/expenses/_expense.html.erb` will be as [given here](/frameworks/ruby/rails-tutorial/chapter-3/view-files.html#content-for-expense-object)

Now need to navigate from `index` page to `new expense` page and vice versa. For that add the navigation links in header so that it will be done. So the code for achieving that is here givn below.

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

and below is the code for `action_nav` partial.

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

Since we have decided to redirect the user to the expenses index page after successful loggin in, we shall modify the `redirect_to` path in sessions_controller's `app/controllers/sessions_controller.rb` create method.

```ruby
redirect_back_or_to expenses_path(), :notice => "Logged in!"
```

Let us try to populate some data using `db/seeds.rb` file. Modify the file like shown below

```ruby
puts "creating default user and admin"
User.create!(username: "user", password: "123123", password_confirmation: "123123")
User.create!(username: "admin", password: "123123", password_confirmation: "123123", role: USER_ROLES[0])

puts "creating expense types"
ExpenseType.create!(name: "Travel")
ExpenseType.create!(name: "Education")
ExpenseType.create!(name: "Health")
```

After modifying the seeds files, we have to populate the records. To do so, run the following command in the terminal

```bash
$ bundle exec rake db:seed
```

##Check Point

Now open your browser and login. You will be redirected to your expenses index page. Then click on `New Expwnse` link. You will see as shown below

![Image for creating new Expense](/images/rails-tutorial/new-expense-creation.png)

Below Image is a screen with some expenses in index page

![Image for Exense index page after creating Expenses](/images/rails-tutorial/expense-index.png)

<a class="button-plain" style="padding: 3px 15px;" href="/frameworks/ruby/rails-tutorial/rails-user-login.html">Prev</a>  <a class="button-plain" style="padding: 3px 15px; float: right;" href="/frameworks/ruby/rails-tutorial/rails-expense-admin-flow.html">Next</a>
