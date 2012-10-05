---
title: Expense Reporting Application with Rails
description: Adding Admin Flow to the Application
tags:
    - ruby
    - rails
    - expense_admin_flow
---

##Introduction

This chapter is continuation for [chapter-3](/frameworks/ruby/rails-tutorial/rails-expense-user-flow.html). This section explains the process how to add the functionality to approve or reject the expense by admin.

##Prerequisites

+ You should complete the [chapter-3](/frameworks/ruby/rails-tutorial/rails-expense-user-flow.html), which lets you to add an expense to the Expense Reporting application.

##Adding Admin Flow to the Application

Admin flow is the functionality left to be added to the application. When a user creates an expense, it will be redirected to an Admin for approval. The Admin can either accept or reject that expense.

When the Admin User successfully logs in, he will be redirected to the index page of expenses. In this page, add one link in the header which will take him to the page showing all pending expenses. Two actions are to be implemented, one action to display the pending expenses and another action to update the status to either accepted or to rejected. Add these two methods in app/controllers/expense_controller.rb as follows. In the `approvals` action `index` template is rendered.

Copy the `expenses_controller` code from [this](/frameworks/ruby/rails-tutorial/code/chapter-4/controller-files.html#content-for-expense-controller) location.

Modify the `app/controllers/application_controller.rb` file code as given below. Here `check_admin_access` is a method which will check whether the current_user is an admin or not. If not, it will redirect him to his expense index page.

```ruby
class ApplicationController < ActionController::Base
  protect_from_forgery

  protected

  def check_admin_access
    redirect_to expenses_path() unless current_user.is_admin?
  end

end
```

Add these two actions `approvals` and `update_status` in the `config/routes.rb` file to generate the routes. Replace the following line in this file

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

A link needs to be added in navigation bar to go to `approvals` page. Modify `app/views/layouts/_actions_nav.html` file as shown below to add this link.

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

To accept/reject an expense, expense details page `app/views/expenses/_expense.html.erb` should have two links. For adding these links, modify the `app/views/expenses/_expense.html.erb` file as [mentioned here](/frameworks/ruby/rails-tutorial/code/chapter-4/view-files.html#code-for-expense-object)

Modify the code in `app/assets/javascripts/application.js` as given below to add some javascript for displaying links.

```js
//= require jquery
//= require jquery_ujs
//= require jquery-ui
//= require_tree .

$(document).ready(function(){
  $('#date-field').datepicker({
    dateFormat: 'dd/mm/yy'
  });

  $('.arrow-down-actions').live('click', function(e) {
    $('.admin-actions :not($(this).siblings("ul.admin-actions"))').hide();
    $(this).siblings('ul').toggle();
  });
})
```

##Check Point

**Run the App Locally**

Execute the following command to run the app

```bash
$ rails s
```

Open browser and login with Admin user credentials, and click on `Approvals` link. Here you will see all the list of expenses that are in pending status.

![Admin index page for Expenses](/images/rails-tutorial/admin-approval-page.png)

Here you can see the status of expenses in users view.

![User index page after admin accepts particular expense](/images/rails-tutorial/expenses-with-different-statuses.png)

## Adding Pagination to the application

[Kaminari](http://railscasts.com/episodes/254-pagination-with-kaminari) gem provides a cleaner implementation of pagination and offers several improved features, so let’s try it in this application. Kaminari is installed in the usual way: first by adding a reference to it in the application’s `Gemfile` and then running `bundle` to make sure that the gem is installed on local system. Add the following line in Gemfile

```ruby
gem "kaminari"
```

To install this gem, run `bundle install` as shown below.

```bash
$ bundle install
```

Kaminari is a Rails Engine and it comes with a number of view files which can be customized to suit our application. Learn more on [Kaminari gem](https://github.com/amatsuda/kaminari). Generate all the files which are needed for using the pagination. Do this by running the following command

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

Replace the generated view files in `app/views/kaminari` with files available [kaminari.zip](/rails-code/kaminari.zip) to customize the styles for the pagination.

Now modify the `app/controllers/expense_conttroller.rb` as [given here](/frameworks/ruby/rails-tutorial/code/chapter-4/controller-files.html#content-for-expense-controller-with-pagination). Here in this file the expense objects are converted into Kaminary objects in `index` and `approvals` actions, so that they can be used to get the expenses by page wise. For example the code 

```ruby
def index
  @expenses = current_user.expenses(:include => :expense_type)
end
```
has changed to

```ruby
def index
  @expenses = current_user.expenses(:include => :expense_type).page(params[:page]).per(10)
end
```

Now add the pagination links at the bottom of the view file `app/views/expense/index.html.erb` as shown below.

```rhtml
<div class="pagination pagination-1">
  <%= paginate @expenses %>
</div>
```

## Check Point

Here is the image with Pagination

![Image for Index page with Pagination](/images/rails-tutorial/index-with-pagination.png)

<a class="button-plain" style="padding: 3px 15px;" href="/frameworks/ruby/rails-tutorial/rails-expense-user-flow.html">Prev</a>  <a class="button-plain" style="padding: 3px 15px; float: right;" href="/frameworks/ruby/rails-tutorial/rails-hosting-application-with-vmc.html">Next</a>
