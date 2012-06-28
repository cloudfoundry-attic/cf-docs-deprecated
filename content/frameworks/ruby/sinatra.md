---
title: Sinatra
description: Sinatra Development with Cloud Foundry
tags:
    - ruby
    - sinatra
    - mysql
---

This is a guide for Sinatra developers who are using Cloud Foundry.
For more information about Ruby and Cloud Foundry, see:

+  [Ruby on Rails](http://rubyonrails.org/)
+  [Getting Started with Cloud Foundry](/getting-started.html)
+  [Ruby Application Development with Cloud Foundry](ruby.html)

To use Cloud Foundry services with Sinatra, you must access the
`VCAP_SERVICES` environment variable, as described in
[Ruby Application Development with Cloud Foundry](/frameworks/ruby/ruby.html#using-cloud-foundry-services).

## Sinatra and Bundler

Currently, Cloud Foundry may not properly detect a Sinatra app if it uses
Bundler.  Cloud Foundry greps for `require "sinatra"` in the `*.rb` files,
and picks that file as the main Sinatra application file.  If you use
Bundler to load Sinatra, Cloud Foundry may not recognize your application.
You can put a comment in your application's main file like this:

```ruby
  # require 'sinatra'   # required for framework detection in cloud foundry...
  require 'rubygems'
  require 'bundler'
  Bundler.require
  ...
```

## Sample Sinatra App using DataMapper and Bundler

### Gemfile

```ruby
   source 'http://rubygems.org'

   gem 'sinatra'
   gem 'data_mapper'

   group :development do
     gem 'dm-sqlite-adapter'
   end

   group :production do
     gem 'dm-mysql-adapter'
   end
```

### sinatra_dm.rb
 The name of this sinatra application will be `sinatra_dm.rb`.

 Note the commented out require of sinatra, which is necessary for proper detection of the application's main file.

```ruby
   # Sample Sinatra app with DataMapper
   # Based on http://sinatra-book.gittr.com/ DataMapper example

   # require 'sinatra'   # required for framework detection in cloud foundry.
   require 'rubygems'
   require 'bundler'
   Bundler.require

   if ENV['VCAP_SERVICES'].nil?
     DataMapper::setup(:default, "sqlite3://#{Dir.pwd}/blog.db")
   else
     require 'json'
     svcs = JSON.parse ENV['VCAP_SERVICES']
     mysql = svcs.detect { |k,v| k =~ /^mysql/ }.last.first
     creds = mysql['credentials']
     user, pass, host, name = %w(user password host name).map { |key| creds[key] }
     DataMapper.setup(:default, "mysql://#{user}:#{pass}@#{host}/#{name}")
   end

   class Post
     include DataMapper::Resource
     property :id, Serial
     property :title, String
     property :body, Text
     property :created_at, DateTime
   end

   DataMapper.finalize
   Post.auto_upgrade!

   get '/' do
     @posts = Post.all(:order => [:id.desc], :limit => 20)
     erb :index
   end

   get '/post/new' do
     erb :new
   end

   get '/post/:id' do
     @post = Post.get(params[:id])
     erb :post
   end

   post '/post/create' do
     post = Post.new(:title => params[:title], :body => params[:body])
     if post.save
       status 201
       redirect "/post/#{post.id}"
     else
       status 412
       redirect '/'
     end
   end
```

### Views

#### Index
```erb
    <!-- views/index.erb -->

    <h1>All Blog Posts</h1>
    <ul>
      <% @posts.each do |post| %>
        <li><a href="/post/<%= post.id %>"><%= post.title %></a></li>
      <% end %>
    <br />
    <a href="/post/new">Create new post</a>
```

#### New
```erb
    <!-- views/new.erb -->

    <h1>Create a new blog post</h1>
    <form action="/post/create" method="POST">
     <p>Title: <input type="text" name="title"></input></p>
     <p>Text: <textarea name="body" rows="10" cols="40"></textarea></p>
     <input type="submit" name="Publish" />
    </form>
```

#### Post
```erb
    <!-- views/post.erb -->

    <h1><%= @post.title %></h1>
    <p><%= @post.body %></p>
    <a href="/">All Posts</a>
```

### Bundle and push.

```bash
   $ bundle package
   $ vmc push --runtime ruby19
```
