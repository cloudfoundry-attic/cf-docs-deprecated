---
title: Expense Report Application with Rails
description: Code for Controllers in Rails Expense Report Application
tags:
    - ruby
    - rails
    - rails-controllers-code
---

##Content for Expense Controller

Here the below given code is for `app/controllers/expense_controlelr.rb` file.

```ruby
class ExpensesController < ApplicationController
  before_filter :require_login, :get_pending_expenses
  before_filter :check_admin_access, :only => [:approvals, :update_status]

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
    @expenses = current_user.expenses.includes(:expense_type)
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

  def approvals
    @expenses = Expense.pending.includes(:expense_type)
    render "index"
  end

  def update_status
    @expense = Expense.find(params[:id])
    @expense.update_attribute(:status, params[:status])
    redirect_to :back
  end

  protected

  def get_pending_expenses
    @pending_expenses = Expense.pending if current_user.is_admin?
  end

end
```

## Content for Expense Controller with Pagination

Here below given code is for `expense_controller.rb` file with added script for pagination

```ruby
class ExpensesController < ApplicationController
  before_filter :require_login, :get_pending_expenses
  before_filter :check_admin_access, :only => [:approvals, :update_status]

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
    @expenses = current_user.expenses.includes(:expense_type).page(params[:page]).per(10)
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

  def approvals
    @expenses = Kaminari.paginate_array(Expense.pending.includes(:expense_type)).page(params[:page]).per(10)
    render "index"
  end

  def update_status
    @expense = Expense.find(params[:id])
    @expense.update_attribute(:status, params[:status])
    redirect_to :back
  end

  protected

  def get_pending_expenses
    @pending_expenses = Expense.pending if current_user.is_admin?
  end

end
```