---
title: Ruby on Rails 3.0
description: Ruby on Rails 3.0 Development with Cloud Foundry
tags:
    - ruby
    - rails
    - mysql
---

This is a guide for Ruby on Rails 3.0 developers who are using Cloud Foundry.
For more information about Ruby and Cloud Foundry, see:

+  [Ruby on Rails](http://rubyonrails.org/)
+  [Getting Started with Cloud Foundry](/getting-started.html)
+  [Ruby Application Development with Cloud Foundry](ruby.html)

Using Cloud Foundry services with Ruby on Rails is the same as using services with Sinatra applications, except that MySQL is recognized automatically when you stage a Rails application on Cloud Foundry. For other Cloud Foundry services, you must access the `VCAP_SERVICES` environment variable, as described in [Ruby Application Development with Cloud Foundry](/frameworks/ruby/ruby.html#using-cloud-foundry-services).

## Rails 3.0.10 on Ruby 1.8 and Ruby 1.9

Rails 3.0 works well on Cloud Foundry. Be sure to use the correct version of the MySQL2 gem.

### Gemfile

```ruby
   # If you use a different database in development, hide it from Cloud Foundry
   group :development do
     gem 'sqlite3'
   end

   # Rails 3.0 requires version less than 0.3 of mysql2 gem
   group :production do
     gem 'mysql2', '< 0.3'
   end
```

### Bundle your application:

```bash
$ bundle package
$ bundle install
```

### Deploy the application:

```bash
$ vmc push
   ...
```

Respond to interactive prompts to deploy and start the application.