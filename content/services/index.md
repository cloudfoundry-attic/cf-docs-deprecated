---
title: Overview
description: Application Services Provided by Cloud Foundry
tags:
    - environment
    - services
    - mysql
    - redis
---

Cloud Foundry has support for the following services:

+ [MySQL](http://www.mysql.com/), the open source relational database.
+ [vFabric Postgres](http://www.vmware.com/products/datacenter-virtualization/vfabric-data-director/features.html), relational database based on PostgreSQL.
+ [MongoDB](http://www.mongodb.org/), the scalable, open, document-based database.
+ [Redis](http://redis.io/), the open key-value data structure server.
+ [RabbitMQ](http://www.rabbitmq.com/), for reliable, scalable, and portable messaging for your applications.

You can run these services locally while you develop your application, or you can use [Micro Cloud Foundry](/infrastructure/micro/installing-mcf.html) to host your application. An advantage when using Micro Cloud Foundry is that your application runs in an environment that is very similar to Cloud Foundry. Also, with Micro Cloud Foundry, you do not have to tool your application to connect to different services for your development and deployment environments.

To access Cloud Foundry services from your application, you first create a service, and then bind it to your application. When your application runs on Cloud Foundry, the environment contains a `VCAP_SERVICES` variable that has information about all the services bound to the application. The content of this variable is a JSON document.

The following example `VCAP_SERVICES` variable contains three bound services, two MySQL databases and a Redis database. The JSON document is formatted for readability.

``` javascript
{ "mysql-5.1" : [
      { "credentials" : {
            "host" : "mysql-node01.us-east-1.aws.af.cm",
            "hostname" : "mysql-node01.us-east-1.aws.af.cm",
            "name" : "d6d665aa69817406d8901cd145e05e3c6",
            "password" : "pzAx0iaOp2yKB",
            "port" : 3306,
            "user" : "uB7CoL4Hxv9Ny",
            "username" : "uB7CoL4Hxv9Ny"
          },
        "label" : "mysql-5.1",
        "name" : "mysql-4f700",
        "plan" : "free",
        "tags" : [ "mysql",
            "mysql-5.1",
            "relational"
          ]
      },
      { "credentials" : {
            "host" : "mysql-node01.us-east-1.aws.af.cm",
            "hostname" : "mysql-node01.us-east-1.aws.af.cm",
            "name" : "db777ab9da32047d99dd6cdae3aafebda",
            "password" : "p146KmfkqGYmi",
            "port" : 3306,
            "user" : "uJHApvZF6JBqT",
            "username" : "uJHApvZF6JBqT"
          },
        "label" : "mysql-5.1",
        "name" : "mysql-f1a13",
        "plan" : "free",
        "tags" : [ "mysql",
            "mysql-5.1",
            "relational"
          ]
      }
    ],
  "redis-2.2" : [
      { "credentials" : {
            "hostname" : "179.30.48.44",
            "name" : "redis-70d8d18a-f184-4ec6-bef4-24cae74a9b8c",
            "node_id" : "redis_node_5",
            "password" : "c7945405-73ef-4f1a-bb64-b9e2383e92d6",
            "port" : 5032
          },
        "label" : "redis-2.2",
        "name" : "redis-a42c1",
        "plan" : "free",
        "tags" : [ "redis",
            "redis-2.2",
            "key-value",
            "nosql"
          ]
      } ]
}
```

The document is a list of service types. Each service type contains a list of the provisioned services bound to the application. Usually, there will be just one instance of a service type, but you can create and bind multiple instances of a service type, as demonstrated by the MySQL instances in this example. Each service is described with a number of parameters, including a name and label, and a credentials object, which contains all of the information your application requires to access the service.

Whatever programming language or framework you use, you need to get the `VCAP_SERVICES` variable from the environment, parse it, and extract the connection information and credentials for the service you want to access. Then, you use this data to connect to the service via the library, module, or driver.

Application development frameworks such as Spring, Grails, and Ruby on Rails promote conventions that make it possible to automate much of the application development process. Where possible, Cloud Foundry tools take advantage of conventions and modify your application's configuration at deployment time to use the Cloud Foundry services bound to the application.
