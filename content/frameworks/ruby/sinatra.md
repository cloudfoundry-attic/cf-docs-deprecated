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

To use Cloud Foundry services with Sinatra, you can access the
`VCAP_SERVICES` environment variable, as described in
[Ruby Application Development with Cloud Foundry](/frameworks/ruby/ruby.html#using-cloud-foundry-services)
or use the `cf-runtime` gem.

## Sinatra and Bundler

Currently when doing a `vmc push`, the vmc gem greps for `require "sinatra"` and  `require "sinatra/base"` in the `*.rb` files in order
to detect the type of Ruby Application this is.

### Classic Sinatra Applications

Classic Sinatra Applications are identified by the presence of `require "sinatra"`. If you use
Bundler to load Sinatra, Cloud Foundry may not recognize your application.
You can put a comment in your application's main file like this:

```ruby
  # require 'sinatra'   # required for sinatra classic detection in cloud foundry...
  require 'rubygems'
  require 'bundler'
  Bundler.require
  ...
```

### Modular Sinatra Apps

Cloud Foundry assumes that if your Sinatra application is in a single file that
requires "sinatra/base" it is a modular Sinatra app and it will have a call to
`run!` as seen in the file below

If you deploy an app with `website.rb`, `Gemfile` and `Gemfile.lock` and using the `vmc` gem version >= `0.3.21`

``` ruby

# website.rb
require "rubygems"
require "sinatra/base"
require "haml"
require "cloudfoundry/environment"

class WebsiteApp < Sinatra::Base

  configure do
    set(:port, CloudFoundry::Environment.port || 4567)
  end

  get "/" do
    return "Hello World"
  end

  run! if __FILE__ == $0

end
```

You will see it being detected:

``` bash

Would you like to deploy from the current directory? [Yn]:
Application Name: www-mon-mon
Detected a Sinatra Application, is this correct? [Yn]:
Application Deployed URL [www-mon-mon.cloudfoundry.com]:
Memory reservation (128M, 256M, 512M, 1G, 2G) [128M]:
How many instances? [1]:
Bind existing services to 'www-mon-mon'? [yN]:
Create services to bind to 'www-mon-mon'? [yN]:
Would you like to save this configuration? [yN]:
Creating Application: OK
Uploading Application:
  Checking for available resources: OK
  Processing resources: OK
  Packing application: OK
  Uploading (6K): OK
Push Status: OK
Staging Application 'www-mon-mon': OK
Starting Application 'www-mon-mon': OK

```

If you have multiple files with classes which derive from Sinatra Base, you need
to add a `config.ru` and run as a `rack` application so you can specify which
classes to run.
Here is an example of what the `config.ru` needs to look like to run Sinatra Apps
OpenTour and Events in a single Cloud Foundry app

```ruby
#config.ru

require './opentour'
require './events'

map('/') { run OpenTour }
map('/events') { run Events }
```

For more details on classic and modular Sinatra Apps please check out
the [Sinatra README](http://www.sinatrarb.com/intro#Modular%20vs.%20Classic%20Style)

## Sample Classic Sinatra App using DataMapper and Bundler

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
