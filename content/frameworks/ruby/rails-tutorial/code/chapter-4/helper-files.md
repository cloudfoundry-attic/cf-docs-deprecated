---
title: Expense Report Application with Rails
description: Code for Helpers in Rails Expense Report Application
tags:
    - ruby
    - rails
    - rails-helpers-code
---

##Code for Application Helper

Below is the code for `app/helpers/application_helper.rb`

```ruby
module ApplicationHelper

  def get_status_class(expense)
    status = expense.status
    case status
    when EXPENSE_STATUS[0]
      ""
    when EXPENSE_STATUS[1]
      "green"
    when EXPENSE_STATUS[2]
      "red"
    end
  end

  def get_status_image(expense)
    status = expense.status
    case status
    when EXPENSE_STATUS[0]
      "question-a"
    when EXPENSE_STATUS[1]
      "right-a"
    when EXPENSE_STATUS[2]
      "cross-a"
    end
  end

  def get_cross_image
    image_tag(asset_path("cross-b.png"), :class => "cross-b-img")
  end

  def form_heading_text
    ['new', 'create'].include?(params[:action]) ? "Add New Expense" : "Edit Expense"
  end

  def submit_button_text
    ['new', 'create'].include?(params[:action]) ? "Add Expense" : "Update Expense"
  end

  def get_pointer
    content_tag(:i, image_tag(asset_path 'arrow-right-a.png', :class => "arrow-right-a-img"),
                :class => "bg-arrow-right-a")
  end
end
```