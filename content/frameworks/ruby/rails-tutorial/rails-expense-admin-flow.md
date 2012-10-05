---
title: Expense Reporting Application with Rails
description: Adding Admin Flow to the Application
tags:
    - ruby
    - rails
    - expense_admin_flow
---

##Introduction

This chapter is continuation for the [chapter-3](/frameworks/ruby/rails-tutorial/rails-expense-user-flow.html), and this section explains the process how to add the functionality to approve or reject the expense by admin.

##Prerequisites

+ You should complete the [chapter-3](/frameworks/ruby/rails-tutorial/rails-expense-user-flow.html), which lets you to add the expense creation to our Expense Reporting application.

##Adding Admin Flow to the Application

Now the only thing remaining is admin flow. When ever a normal user creates an expense, it will redirect to the admin for approval. The admin can either accept or reject that expense.

When a user successfully login, then he will be redirected to the index page of expenses, here in this page, a normal user will be able to see only his expenses, where as an admin shoulb be able to see the expenses created by all users.

So we are going to modify the `index` action in `app/controllers/expense_controller.rb` as follows.

```ruby
def index
  if current_user.is_admin?
    @expenses = Expense.all
  else
    @expenses = current_user.expenses
  end
end
```

Now admin user will be able to see all the expenses in his index page. Now we need to make the actions for accept or reject the expense. Add a method `update_status` in `app/controllers/expense_controller.rb` file.

```ruby
def update_status
  @expense = Expense.find(params[:id])
  @expense.update_attributes(:status => params[:status])
  redirect_to :back
end
```

add the method name in routes file `config/routes.rb`. In this file replace `resources :expenses` with the following.

```ruby
resources :expenses do
  member do
    put :update_status
  end
end
```

Now we need to give the links in expense details page for accept/reject. So modify the `app/views/expenses/_expense.html.erb` file as follows.

```rhtml
<tr>
  <td><%= I18n.localize(expense.date, :format => "%d-%I-%Y") if expense.date %></td>
  <td><%= expense.expense_type.name %></td>
  <td><%= expense.description %></td>
  <td><%= expense.status %></td>
  <td>
    <% if current_user.is_admin? %>
      <%= link_to "Approve", update_status_expense_path(:id => expense.id, :status => EXPENSE_STATUS[1]), :method => :put %> | 
      <%= link_to "Reject", update_status_expense_path(:id => expense.id, :status => EXPENSE_STATUS[2]), :method => :put %>
    <% else %>
    <% else %>
      <%= link_to "Edit", edit_expense_path(expense) %> | 
      <%= link_to "Delete", expense_path(expense), :confirm => "Are you sure want to delete?", :method => :delete %>
    <% end %>
  </td>
</tr>
```
Thats all, now open browser and login as an admin, you will see a list of all expenses created by all the users with the actions to accept or reject the expense.

![Admin index page for Expenses](/images/admin-index-page.png)

Here each expense will be displayed with its status whether pending or accepted or rejected. Just click on accept, then the status will be changed to `accepted`. 

![Admin index page after accepting an expense](/images/admin-action-result.png)

Here you can see the status of the expense in users view.

![User index page after admin accepts particular expense](/images/user-index-page-after-admin-approval.png)

[![Prev](/images/prev.png)](/frameworks/ruby/rails-tutorial/rails-expense-user-flow.html)  <span style="float: right;">[![Next](/images/next.png)](/frameworks/ruby/rails-tutorial/rails-hosting-application-with-vmc.html)</span>
