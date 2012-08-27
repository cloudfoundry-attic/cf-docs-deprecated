---
title: Ruby on Rails 3.1 and Above
description: Ruby on Rails 3.1 Development with Cloud Foundry
tags:
    - ruby
    - rails
    - mysql
---

This is a guide for Rails 3.1 and 3.2 developers using Cloud Foundry. It makes the following assumptions:

+   You have vmc installed.

+   You are proficient in developing Rails applications and are familiar with tools used to manage dependencies in Rails applications.

For more information about Ruby and Cloud Foundry, see:

+  [Ruby on Rails](http://rubyonrails.org/)
+  [Getting Started with Cloud Foundry](/getting-started.html)
+  [Ruby Application Development with Cloud Foundry](ruby.html)

Using Cloud Foundry services with Ruby on Rails is the same as using services with Sinatra applications, except that MySQL is recognized automatically when you stage a Rails application on Cloud Foundry. For other Cloud Foundry services, you must access the `VCAP_SERVICES` environment variable, as described in [Ruby Application Development with Cloud Foundry](/frameworks/ruby/ruby.html#using-cloud-foundry-services).

## Rails 3.1.2 and 3.2 on Ruby 1.8 and Ruby 1.9

Rails 3.1 introduces the asset pipeline. To get the asset pipeline working on Cloud Foundry, precompile your assets in your development environment, which compiles them into `public/assets`, then tweak the production environment configuration before excuting a normal `vmc push`.

The following steps illustrate the process.

### Gemfile

#### Ruby 1.8

```ruby

   # If you use a different database in development, hide it from Cloud Foundry.
   group :development do
     gem 'sqlite3'
   end

   # Rails 3.1 can use the latest mysql2 gem.
   group :production do
     gem 'mysql2'
   end

```

#### Ruby 1.9

```ruby

   # If you use a different database in development, hide it from Cloud Foundry.
   group :development do
     gem 'sqlite3'
   end

   # Rails 3.1 can use the latest mysql2 gem.
   group :production do
     gem 'mysql2'
   end

   # For Ruby 1.9 Cloud Foundry requires a tweak to the jquery-rails gem.
   # gem 'jquery-rails'
   gem 'cloudfoundry-jquery-rails'

   # For Ruby 1.9 Cloud Foundry requires a tweak to devise.
   # Uncomment next line if you plan to use devise.
   # gem 'cloudfoundry-devise', :require => 'devise'

```

### Bundle your application:

```bash
$ bundle package
$ bundle install
```

### Configs

Edit `config/environments/production.rb` and change

```ruby
config.serve_static_assets = false
```

to

```ruby
config.serve_static_assets = true
```

### Assets

Pre-compile your asset pipeline.

```bash
$ bundle exec rake assets:precompile
```

### VCS

Commit the current configuration to your version control system. Consider including:
   + `Gemfile.lock`
   + gems packaged into `vendor/cache`
   + assets compiled into `public/assets`

### Deploy

```bash
$ vmc push
```

Respond to the interactive prompts to deploy and start the application.
