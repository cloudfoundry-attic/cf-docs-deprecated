---
title: Expense Reporting Application with Rails
description: Code for Views in Rails Expense Reporting Application
tags:
    - ruby
    - rails
    - rails-view-code
---

##Content for Expense Form

Listing below is the code snippet for `app/views/expenses/_form.html.erb` partial file. Before adding this code make sure that you have added methods in [application_helper.rb](/frameworks/ruby/rails-tutorial/code/chapter-4/helper-files.html#code-for-application-helper)

```rhtml
<%= form_for @expense, :html => {:multipart => true, :class => "mtop80"} do |f| %>
  <% if @expense.errors.any? %>
    <div class="error_messages">
      <ul>
        <% for message in @expense.errors.full_messages %>
          <li><%= message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

 <div class="action-a">
    <%= f.label :description, :class => "label-name" %>
    <%= f.text_area :description, :class => "textbox-a", :placeholder => "description" %>
  </div>

  <div class="action-a">
    <%= f.label :amount, :class => "label-name" %>
    <%= f.text_field :amount, :class => "textbox-a dd-b flt-left mleft0", :style => "width: 185px;", :placeholder => "Amount like 210.50" %>

    <a class="label-7 flt-right" href="javascript:void(0)" id="attachment-holder"><img class="clip-a-img" src="<%= asset_path 'clip-a.png' %>"><span id="attachment-name">Attach Receipt</span> </a>
    <%= f.file_field :attachment, :style => "display: none;" %>
  </div>

  <div class="action-a">
    <%= f.label :expense_type_id, :class => "label-name" %>
    <%= collection_select(:expense, :expense_type_id, ExpenseType.all, :id, :name, options ={:prompt => "Select Expense Type"}, html_options = {:class =>"textbox-a dd-a mleft0"} ) %>

  </div>

  <div class="action-a">
    <%= f.label :date, "Generated on", :class => "label-name" %>
    <input type="text" name="expense[date]" placeholder="DD/MM/YY" id="date-field" class="textbox-a dd-b mleft0" style="width: 185px;" />
  </div>
  <%= f.submit submit_button_text, :class => "btn-a add-expense-btn" %>
<% end %>

<script type="text/javascript">
  $(document).ready(function(){
    $('#attachment-holder').live('click', function(){
      $('#expense_attachment').trigger('click');
    });

    $("input[type='file']").change( function() {
      if($(this).val() == "")
        $('#attachment-name').html("Attach Receipt");
      else
        $('#attachment-name').html($(this).val().replace(/^.*\\/g, ''));
    });
  })
</script>
```

##Content for Expense Object

Here the below given code id for `app/views/expenses/_expense.html.erb` file. Before adding this code make sure that you have added methods in [application_helper.rb](/frameworks/ruby/rails-tutorial/code/chapter-4/helper-files.html#code-for-application-helper)

```rhtml
<tr>
  <td>
    <h5 class="label-4"><%= I18n.localize(expense.date, :format => "%d-%I-%Y") if expense.date %></h5>
  </td>
  <td>
    <h5 class="label-4"><%= truncate(expense.expense_type.name, :length => 20) %></h5>
  </td>
  <td class="fixed-width">
    <h5 class="label-4">
      <%= truncate(expense.description, :length => 50) %>
      <% if expense.attachment? %>
        <%= link_to image_tag(asset_path('clip-b.png'), :class => 'clip-b-img flt-right'), expense.attachment.url, :target => "_blank" %>
      <% end %>
    </h5>
  </td>
  <td>
    <h5 class="label-4">
      <i class="alt-a label-5 <%= get_status_class(expense) %>">
        <img class="<%= get_status_image(expense) %>-img" src='<%= asset_path "#{get_status_image(expense)}.png" %>'>
        <%= expense.status %>
      </i>

      <i class="alt-b label-6">
        <img class="<%= get_status_image(expense) %>-img" src='<%= asset_path "#{get_status_image(expense)}.png" %>'> <%= expense.status %>
        <%= link_to(edit_expense_path(expense), :class => "bg-edit-b") do %>
          <img class="clip-b-img" src="<%= asset_path 'edit-a.png' %>">
        <% end %>
        <%= link_to get_cross_image.html_safe, expense_path(expense), :class => "bg-cross-b", :confirm => "Are you sure want to delete?", :method => :delete, %>
      </i>
    </h5>
  </td>
</tr>
```