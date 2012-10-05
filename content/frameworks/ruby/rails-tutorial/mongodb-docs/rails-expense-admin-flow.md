---
title: Expense Reporting Application with Rails
description: Adding Admin Flow to the Application
tags:
    - ruby
    - rails
    - expense_admin_flow
---

##Introduction

This chapter is continuation for the [chapter-3](/frameworks/ruby/rails-tutorial/mongodb-docs/rails-expense-user-flow.html), and this section explains the process how to add the functionality to approve or reject the expense by admin.

##Prerequisites

+ You should complete the [chapter-3](/frameworks/ruby/rails-tutorial/mongodb-docs/rails-expense-user-flow.html), which lets you to add the expense creation to our Expense Reporting application.

##Adding Admin Flow to the Application

Now the only thing remaining is admin flow. When ever a user creates an expense, it will redirect to the admin for approval. The admin can either accept or reject that expense.

When the Admin User successfully login, then he will be redirected to the index page of expenses. Here in this page, we need to add one link in the header so that on clicking on this link, it takes him to the page where all the pending expenses will be shown, so that he can approve or reject.

So now we need one action to display the pending expenses and one action to update the status to either to `accepted` or to `rejected`. Add these two methods in `app/controllers/expense_controller.rb` as follows. Here in `approvals` action we are rendering the `index` template.

Get the code for [expense_controller here](/frameworks/ruby/rails-tutorial/code/chapter-4/controller-files.html#content-for-expense-controller)

Now modify the `app/controllers/application_controller.rb` file code as given below

```ruby
class ApplicationController < ActionController::Base
  protect_from_forgery

  protected

  def check_admin_access
    redirect_to expenses_path() unless current_user.is_admin?
  end

end
```

Now we need to add these two actions `approvals` and `update_status` in the `config/routes.rb` file so that the routes to these actions will be formed. So in this file, replace

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

Now we need to add the link in navigation bar to go to `approvals` page. Modify `app/views/layouts/_actions_nav.html` file as shown below

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

Now we need to give the links in expense details page for accept/reject. So modify the `app/views/expenses/_expense.html.erb` file as [mentioned here](/frameworks/ruby/rails-tutorial/code/chapter-4/view-files.html#code-for-expense-object)

Now modify the code in `app/assets/javascripts/application.js` as given below.

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

Thats all, now open browser and login as an admin, and click on `Approvals` link. Here you will see all the list of expenses that are in pending state

![Admin index page for Expenses](/images/rails-tutorial/admin-approval-page.png)

Here you can see the status of the expense in users view.

![User index page after admin accepts particular expense](/images/rails-tutorial/expenses-with-different-statuses.png)

## Adding Pagination to the application

[Kaminari](http://railscasts.com/episodes/254-pagination-with-kaminari) gem provide a cleaner implementation of pagination and offers several improved features, so let’s try it in our application. Kaminari is installed in the usual way: first by adding a reference to it in the application’s `Gemfile` and then running `bundle` to make sure that the gem is installed on our system. So add the following line in your Gemfile

```ruby
gem "kaminari"
```

And now run bundle install to install this gem

```bash
$ bundle install
```

Kaminari is a Rails Engine and it comes with a number of view files which can be customized to suit our application. Generate all the files which are needed for using the pagination. Do this by running the following command

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

Replace the view files in `app/views/kaminari` generated by default with the [files available here](/rails-code/kaminari.zip) to customize the styles for the pagination.

Now modify the `app/controllers/expense_conttroller.rb` as [given here](/frameworks/ruby/rails-tutorial/code/chapter-4/controller-files.html#content-for-expense-controller-with-pagination)

Now add the pagination links at the bottom of the view file `app/views/expense/index.html.erb` as shown below.

```rhtml
<div class="pagination pagination-1">
  <%= paginate @expenses %>
</div>
```

## Check Point

Here is the image with Pagination

![Image for Index page with Pagination](/images/rails-tutorial/index-with-pagination.png)

<a class="button-plain" style="padding: 3px 15px;" href="/frameworks/ruby/rails-tutorial/mongodb-docs/rails-expense-user-flow.html">Prev</a>  <a class="button-plain" style="padding: 3px 15px; float: right;" href="/frameworks/ruby/rails-tutorial/mongodb-docs/rails-hosting-application-with-vmc.html">Next</a>
