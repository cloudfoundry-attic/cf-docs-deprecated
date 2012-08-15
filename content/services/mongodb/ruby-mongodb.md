---
title: Using MongoDB with Ruby
description: Ruby Development with the MongoDB Service
tags:
    - mongodb
    - sinatra
    - ruby
---

[MongoDB](http://www.mongodb.org), the scalable, open source, document-oriented database is provided as a service on Cloud Foundry.
This section describes how you can adapt a Ruby on Rails application to access the Cloud Foundry mongodb service.
It is assumed that you are using the [MongoMapper](http://mongomapper.com) ORM.

### Gemfile

``` bash
$ gem install "mongo_mapper"
$ gem install "bson_ext"
```

Add BSON for serialization of JSON-like documents; it is needed to interface with the MongoDB Ruby driver.

### Rails
If your application is a Rails app, modify the production section of `config/mongo.yml` to set credentials, host, and port for Cloud Foundry by parsing the JSON-formatted `VCAP_SERVICES` environment variable.

``` erb
production:
host: <%= JSON.parse(ENV['VCAP_SERVICES'])['mongodb-2.0'].first['credentials']['hostname'] rescue 'localhost' %>
port: <%= JSON.parse( ENV['VCAP_SERVICES'] )['mongodb-2.0'].first['credentials']['port'] rescue 27017 %>
database:  <%= JSON.parse( ENV['VCAP_SERVICES'] )['mongodb-2.0'].first['credentials']['db'] rescue 'tutorial_db' %>
username: <%= JSON.parse( ENV['VCAP_SERVICES'] )['mongodb-2.0'].first['credentials']['username'] rescue '' %>
password: <%= JSON.parse( ENV['VCAP_SERVICES'] )['mongodb-2.0'].first['credentials']['password'] rescue '' %>

```

If your application is not a Rails app, the `JSON.parse()` code in the previous step demonstrates how to extract from `VCAP_SERVICES` the information you need to construct a [MongoDB connection string](http://www.mongodb.org/display/DOCS/Connections) to access the Cloud Foundry MongoDB service.

### Bundle

```bash
$ bundle package
$ bundle install
```

### Deploy

When vmc asks if you want to bind any services, enter `y` and choose `mongodb` from the menu. Provide a name for the service or accept the default, as in this transcript:

``` bash
$ vmc push --runtime ruby19
    Would you like to deploy from the current directory? [Yn]:
    Application Name: test
    Application Deployed URL [test.pubs.cloudfoundry.me]:
    Detected a Sinatra Application, is this correct? [Yn]:
    Memory Reservation (64M, 128M, 256M, 512M, 1G) [256M]:
    Creating Application: OK
    Would you like to bind any services to 'test'? [yN]: y
    Would you like to use an existing provisioned service [yN]? N
    The following system services are available
    1: mongodb
    2: mysql
    3: postgresql
    4: rabbitmq
    5: redis
    Please select one you wish to provision: 1
    Specify the name of the service [mongodb-dcc48]:
    Creating Service: OK
    Binding Service [mongodb-dcc48]: OK
    Uploading Application:
      Checking for available resources: OK
      Processing resources: OK
      Packing application: OK
      Uploading (8K): OK
    Push Status: OK
    Staging Application: OK
    Starting Application: OK
```
