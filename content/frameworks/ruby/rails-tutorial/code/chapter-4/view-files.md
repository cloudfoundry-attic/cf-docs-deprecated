---
title: Expense Reporting Application with Rails
description: Code for Views in Rails Expense Reporting Application
tags:
    - ruby
    - rails
    - rails-views-code
---

##Content for Expense Object

Here the below given code is for app/views/expenses/_expense.html.erb file. Before modifying this code make sure that you have added methods in [application_helper.rb](/frameworks/ruby/rails-tutorial/code/chapter-4/helper-files.html#code-for-application-helper)

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
				<% if current_user.is_admin? && params[:action].eql?("approvals") %>
					<span>
						<img class="arrow-down-b-img arrow-down-actions" src="<%= asset_path 'arrow-down-b.png' %>">
						<ul class="admin-actions" style="display: none;">
							<li>
								<img class="right-b-img" src="<%= asset_path 'right-a.png' %>">
								<%= link_to "Approve", update_status_expense_path(:id => expense.id, :status => EXPENSE_STATUS[1]), :method => :put %>
							</li>
							<li>
								<img class="cross-a-img" src="<%= asset_path 'cross-a.png' %>">
								<%= link_to "Reject", update_status_expense_path(:id => expense.id, :status => EXPENSE_STATUS[2]), :method => :put %>
							</li>
						</ul>
					</span>
				<% else %>
					<%= link_to(edit_expense_path(expense), :class => "bg-edit-b") do %>
						<img class="clip-b-img" src="<%= asset_path 'edit-a.png' %>">
					<% end %>
					<%= link_to get_cross_image.html_safe, expense_path(expense), :class => "bg-cross-b", :confirm => "Are you sure want to delete?", :method => :delete, %>
				<% end %>
			</i>

		</h5>
	</td>
</tr>
```