---
title: Using MySQL with Ruby
description: Ruby Development with the MySQL Service
tags:
    - ruby
    - sinatra
    - mysql
---

Ruby on Rails applications that use MySQL are reconfigured automatically when you deploy to Cloud Foundry.
This section describes how to access the Cloud Foundry MySQL service from other Ruby applications, such as applications that use Sinatra.

Install the mysql2 gem:

```bash
$ gem install mysql2
```

List the gem for the database as a dependency in your application's Gemfile:

``` ruby
gem "mysql2"
```

Rails 3.0 requires mysql2 version less than 0.3.

``` ruby
gem "mysql2", "< 0.3"
```

In your application, get the connection information from the environment and connect to the database:

``` ruby
configure do
    services = JSON.parse(ENV['VCAP_SERVICES'])
    mysql_key = services.keys.select { |svc| svc =~ /mysql/i }.first
    mysql = services[mysql_key].first['credentials']
    mysql_conf = {:host => mysql['hostname'], :port => mysql['port'],
        :username => mysql['user'], :password => mysql['password']}
    @@client = Mysql2::Client.new mysql_conf
end
```

Before pushing the application to the cloud, execute these commands:

```bash
$ bundle package
$ bundle install
```

Target Cloud Foundry and log in with your Cloud Foundry credentials.

```bash
$ vmc target api.cloudfoundry.com
$ vmc login
```

Push your application to Cloud Foundry. During the initial push, create the mysql Cloud Foundry service and bind it to your application, as shown in this `vmc push` transcript:

```bash
$ vmc push
    Would you like to deploy from the current directory? [Yn]:
    Application Name: myapp
    Application Deployed URL [myapp.cloudfoundry.com]:
    Detected a Sinatra Application, is this correct? [Yn]:
    Memory Reservation (64M, 128M, 256M, 512M, 1G) [128M]:
    Creating Application: OK
    Would you like to bind any services to 'myapp'? [yN]: y
    The following system services are available
    1: atmos
    2: mongodb
    3: mysql
    4: postgresql
    5: rabbitmq
    6: redis
    Please select one you wish to provision: 3
    Specify the name of the service [mysql-be3ac]:
    Creating Service: OK
    Binding Service [mysql-be3ac]: OK
    Uploading Application:
      Checking for available resources: OK
      Processing resources: OK
      Packing application: OK
      Uploading (219K): OK
    Push Status: OK
    Staging Application: OK
    Starting Application: OK

```
