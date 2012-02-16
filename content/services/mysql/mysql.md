---
title: MySQL
description: Introduction to the Cloud Foundry MySQL Service
tags:
    - mysql
    - overview
---

[MySQL](http://www.mysql.com/products/community), the popular open source relational database, is provided as a service on Cloud Foundry, accessible to applications using any of the Cloud Foundry supported runtimes and frameworks. When deploying a Rails, Grails, or Spring application to Cloud Foundry, vmc or STS may automatically configure your application to use the Cloud Foundry instance. If your application cannot be automatically configured, you can get the information your application needs to connect to the database from the [`VCAP_SERVICES`](#the-vcap_services-environment-variable) environment variable.

## The VCAP_SERVICES Environment Variable

If your application cannot be automatically configured, you must provide code to get connection information from the `VCAP_SERVICES` variable, which is set in the application's Cloud Foundry environment. The contents of this variable is a JSON document containing a list of all provisioned services bound to the application.

Following is an example JSON document (reformatted for readability) from the environment of a Cloud Foundry application with two provisioned MySQL services.

``` javascript
{"mysql-5.1":[
    {
        "name":"mysql-4f700",
        "label":"mysql-5.1",
        "plan":"free",
        "tags":["mysql","mysql-5.1","relational"],
        "credentials":{
            "name":"d6d665aa69817406d8901cd145e05e3c6",
            "hostname":"mysql-node01.us-east-1.aws.af.cm",
            "host":"mysql-node01.us-east-1.aws.af.cm",
            "port":3306,
            "user":"uB7CoL4Hxv9Ny",
            "username":"uB7CoL4Hxv9Ny",
            "password":"pzAx0iaOp2yKB"
        }
    },
    {
        "name":"mysql-f1a13",
        "label":"mysql-5.1",
        "plan":"free",
        "tags":["mysql","mysql-5.1","relational"],
        "credentials":{
            "name":"db777ab9da32047d99dd6cdae3aafebda",
            "hostname":"mysql-node01.us-east-1.aws.af.cm",
            "host":"mysql-node01.us-east-1.aws.af.cm",
            "port":3306,
            "user":"uJHApvZF6JBqT",
            "username":"uJHApvZF6JBqT",
            "password":"p146KmfkqGYmi"
        }
    }
]}
```

Retrieve the value of `VCAP_SERVICES` using your programming language's facility for accessing OS environment variables. In Java, for example, it is `java.lang.System.getenv("VCAP_SERVICES")`; in  Ruby, `ENV['VCAP_SERVICES']`. In Node.js (JavaScript), use `process.env.VCAP_SERVICES` and in Python it is `os.getenv("VCAP_SERVICES")`.

Use a language-specific JSON library or module to parse the value and access the information you need.

The value of the `name` key can be used to distinguish between multiple MySQL instances; it is set when you create the instance using vmc or STS.

The `credentials` object contains all of the data you need to connect to MySQL through a driver or library.

+ `hostname` and `host` have the same value, which is the host where the MySQL server is running
+ `port` is the port where MySQL server accepts connections on the host
+ `user` and `username` are the name of the MySQL database user
+ `password` is the user's MySQL password
+ `name` is the name of the MySQL database

Look for language- and framework-specific examples using MySQL in the [Services](/services.html) section of the Cloud Foundry Getting Started Website.