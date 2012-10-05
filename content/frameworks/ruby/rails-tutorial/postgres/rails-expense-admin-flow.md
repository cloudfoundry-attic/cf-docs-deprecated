---
title: Expense Report Application with Rails and PostgreSQL
description: Adding Admin Flow to the Application
tags:
    - ruby
    - rails
    - expense-admin-flow
---

##Introduction

This chapter is continuation of [chapter 3](/frameworks/ruby/rails-tutorial/postgres/rails-expense-user-flow.html). This section explains how to add the functionality that allows the admin to approve or reject expenses.

##Prerequisites

+ You should have completed [chapter 3](/frameworks/ruby/rails-tutorial/postgres/rails-expense-user-flow.html) that lets you to add expenses to the Expense Report application.

##Setup: Exercise4-Starter

If you are starting this chapter from the downloaded source code [Exercise4-Starter.zip](/rails-code/expense-report-postgres/Exercise4-Starter.zip), please setup the application as described [here](/frameworks/ruby/rails-tutorial/postgres/psql-starters-guide.html) before continuing with this tutorial.

##Adding Admin Flow to the Application

Admin flow is the only functionality left to be added to the application. When a user creates an expense, it will be redirected to an Admin for approval. The Admin can either accept or reject that expense.

When the Admin User successfully logs in, he will be redirected to the index page of expenses. In this page, add a link in the header which will take him to the page showing all pending expenses. Two actions need to be implemented, one to display the pending expenses and another to update it's status to either accepted or to rejected. Add these two methods in `app/controllers/expense_controller.rb` as shown. In the `approvals` action `index` template is rendered.

Copy the `expenses_controller` code from [this](/frameworks/ruby/rails-tutorial/code/chapter-4/controller-files.html#content-for-expense-controller) location.

Modify the `app/controllers/application_controller.rb` file code as given below. `check_admin_access` is a method which will check whether the current_user is an admin or not. If not, he is redirected to his expenses index page.

```ruby
class ApplicationController < ActionController::Base
  protect_from_forgery

  protected

  def check_admin_access
    redirect_to expenses_path() unless current_user.is_admin?
  end

end
```

Add the `approvals` and `update_status` actions in the `config/routes.rb` file to generate the routes. Replace the following line in this file

```ruby
resources :expenses
```
with the following piece of code

```ruby
resources :expenses do
  member do
    put :update_status
  end
  collection do
    get :approvals
  end
end
```

Modify `app/views/layouts/_actions_nav.html` file as shown below to add a link in the navigation bar that will direct to the 'approvals' page.

```rhtml
<div id="nav">
  <ul>
    <% if current_user.is_admin? %>
      <li class="<%= params[:action].eql?('approvals') ? 'selected' : '' %>">
        <%= link_to "Approvals (#{@pending_expenses.count})", approvals_expenses_path() %>
      </li>
    <% end %>
    <li class="<%= params[:action].eql?('index') ? 'selected' : '' %>">
      <%= link_to "My Expenses", expenses_path %>
    </li>
    <li class="<%= ['index', 'approvals'].include?(params[:action]) ? '' : 'selected' %>">
      <%= link_to "New Expense", new_expense_path %>
    </li>
  </ul>
</div>
```

In order to accept/reject an expense, expense details page `app/views/expenses/_expense.html.erb` should have two links. To add these links, modify the `app/views/expenses/_expense.html.erb` file as [mentioned here](/frameworks/ruby/rails-tutorial/code/chapter-4/view-files.html#code-for-expense-object)

Modify the `app/views/expenses/index.html.erb` file as shown below to display the `username` to know by which user a particular expense has been created.

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
    <% if params[:action].eql?("approvals") %>
      <td>
        <h5 class="label-3">Created By</h5>
      </td>
    <% end %>
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

Modify the code in `app/assets/javascripts/application.js` as shown below to add javascript to display links.

```js
//= require jquery
//= require jquery_ujs
//= require jquery-ui
//= require_tree .

$(document).ready(function(){
  $('#date-field').datepicker({
    dateFormat: 'dd/mm/yy',
    maxDate: +0
  });

  $('.arrow-down-actions').live('click', function(e) {
    $('.admin-actions :not($(this).siblings("ul.admin-actions"))').hide();
    $(this).siblings('ul').toggle();
  });
})
```

##Setup: Exercise4-Complete

If you want to test the application from the downloaded source code [Exercise4-Complete.zip](/rails-code/expense-report-postgres/Exercise4-Complete.zip), please setup the application as described [here](/frameworks/ruby/rails-tutorial/postgres/psql-completers-guide.html) before you proceed to *check point*.

##Check Point

**Run the App Locally**

Execute the following command to run the app

```bash
$ rails s
```

Open your browser and login with Admin user credentials and click on `Approvals` link. You will see a list of pending expenses.

![Admin index page for Expenses](/images/screenshots/rails/postgres/rails-expense-admin-flow/admin-approval-page.png)

Here you can see the status of expenses in users view.

![User index page after admin accepts particular expense](/images/screenshots/rails/postgres/rails-expense-admin-flow/expenses-with-different-statuses.png)

## Adding Pagination to the application

[Kaminari](http://railscasts.com/episodes/254-pagination-with-kaminari) gem provides a cleaner implementation of pagination and offers several improved features, so let’s try it in this application. Kaminari is installed in the usual way: first by adding a reference to it in the application’s `Gemfile` and then running `bundle` to make sure that the gem is installed on local system. Add the following line in Gemfile

```ruby
gem "kaminari"
```

To install this gem, run `bundle install` as shown below.

```bash
$ bundle install
```

Kaminari is a Rails Engine and it comes with a number of view files that can be customized to suit our application. Learn more on Kaminari gem [here](https://github.com/amatsuda/kaminari). Generate all the files which are needed for using the pagination. Do this by running the following command

```bash
$ rails g kaminari:views default
    create  app/views/kaminari/_current_page.html.erb
    create  app/views/kaminari/_first_page_link.html.erb
    create  app/views/kaminari/_last_page_link.html.erb
    create  app/views/kaminari/_next_link.html.erb
    create  app/views/kaminari/_next_span.html.erb
    create  app/views/kaminari/_page_link.html.erb
    create  app/views/kaminari/_paginator.html.erb
    create  app/views/kaminari/_prev_link.html.erb
    create  app/views/kaminari/_prev_span.html.erb
    create  app/views/kaminari/_truncated_span.html.erb
```

Replace the generated view files in `app/views/kaminari` with the ones available [kaminari.zip](/rails-code/kaminari.zip) to customize the styles for the pagination.

Now modify the `app/controllers/expense_conttroller.rb` as [given here](/frameworks/ruby/rails-tutorial/code/chapter-4/controller-files.html#content-for-expense-controller-with-pagination). Here in this file the expense objects are converted into Kaminari objects in `index` and `approvals` actions, so they can be used to view expenses by page by page. For example the code

```ruby
def index
  @expenses = current_user.expenses.includes(:expense_type)
end
```
has changed to

```ruby
def index
  @expenses = current_user.expenses.includes(:expense_type).page(params[:page]).per(10)
end
```

Now add the pagination links at the bottom of the view file `app/views/expenses/index.html.erb` as shown below.

```rhtml
<div class="pagination pagination-1">
  <%= paginate @expenses %>
</div>
```

## Check Point

Here is an image with Pagination

![Image for Index page with Pagination](/images/screenshots/rails/postgres/rails-expense-admin-flow/index-with-pagination.png)

<a class="button-plain" style="padding: 3px 15px;" href="/frameworks/ruby/rails-tutorial/postgres/rails-expense-user-flow.html">Prev</a>  <a class="button-plain" style="padding: 3px 15px; float: right;" href="/frameworks/ruby/rails-tutorial/postgres/rails-hosting-application-with-vmc.html">Next</a>
