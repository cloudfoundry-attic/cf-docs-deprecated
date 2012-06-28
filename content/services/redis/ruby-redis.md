---
title: Using Redis with Ruby
description: Ruby Application Development with the Redis Service
tags:
    - ruby
    - redis
---

[Redis](http://redis.io/) is an open source key-value store, also known as a NoSQL database. You set, get, update and delete information using a key.

For application written in Ruby, such as applications in Rails or Sinatra, follow this process:

*  Add the gem to your application's Gemfile.

``` ruby
gem 'redis'
```

Load the library into your application's runtime. In Rails, for instance, use the require statement in application.rb.
In Sinatra, add this to your .rb configuration file:

``` ruby
require 'redis'
```

Configure your environment to locate the Redis service on the cloud:

``` ruby
configure do
    services = JSON.parse(ENV['VCAP_SERVICES'])
    redis_key = services.keys.select { |svc| svc =~ /redis/i }.first
    redis = services[redis_key].first['credentials']
    redis_conf = {:host => redis['hostname'], :port => redis['port'], :password => redis['password']}
    @@redis = Redis.new redis_conf
end
```

Redis credentials are stored in the JSON-formatted VCAP_SERVICES environment variable.

The last line creates a class variable @@redis, available
to all subclasses in your application, that will be used at
runtime to add keys/values to Redis.

In your application use [Redis commands](http://redis.io/commands) to edit and add key/values to the data store.

Run `bundle package` and update or add your application to the cloud:

```bash
$ bundle package
$ vmc stop appname
$ vmc update
```

or, to add:

```bash
$ bundle package
$ vmc push
```

Create the Redis service and bind the application to it.

```bash
$ vmc create-service redis --bind appname
```

For updated applications, start again:

```bash
$ vmc start appname
```
