---
title: Accessing Cloud Foundry Services
description: Accessing Cloud Foundry Services
tags:
    - vmc
    - mongodb
---

## Introduction
This section will teach you how to access the MongoDB service on Cloud Foundry using tunneling.

## Prerequisites
Before you begin this tutorial, you should:

1. Have installed [Caldecott](/tools/vmc/caldecott.html).
2. Have installed [STS](http://www.springsource.org/spring-tool-suite-download) or Eclipse.
3. Have installed [MongDB](http://www.mongodb.org/).

## Accessing MongoDB on Cloud Foundry via vmc Tunneling

**Step 1**  Point vmc target to `http://api.cloudfoundry.com` and login to Cloud Foundry using your credentials.

```bash
$ vmc target http://api.cloudfoundry.com
Successfully targeted to [http://api.cloudfoundry.com]

$ vmc login
Attempting login to [http://api.cloudfoundry.com]
Email: *************
Password: **********
Successfully logged into [http://api.cloudfoundry.com]
```

**Step 2** Now enter **vmc services** command to list the services you have created.

```bash
$ vmc services

============== System Services ==============

+------------+---------+---------------------------------------+
| Service    | Version | Description                           |
+------------+---------+---------------------------------------+
| mongodb    | 2.0     | MongoDB NoSQL store                   |
| mysql      | 5.1     | MySQL database service                |
| postgresql | 9.0     | PostgreSQL database service (vFabric) |
| rabbitmq   | 2.4     | RabbitMQ message queue                |
| redis      | 2.2     | Redis key-value store service         |
+------------+---------+---------------------------------------+

=========== Provisioned Services ============

+----------+------------+
| Name     | Service    |
+----------+------------+
| mongodb1 | mongodb    |
+----------+------------+

```

**Step 3** To get the remote service connection details, enter **vmc tunnel data service name**

```bash
$ vmc tunnel mongodb1
Getting tunnel connection info: OK

Service connection info:
  username : u2a13abe8b0aa47b39ade9fd886536a05
  password : pa2f356dc4457452abaabd71217b5113e
  name     : db
  url      : mongodb://u2a13abe8b0aa47b39ade9fd886536a05:pa2f356dc4457452abaabd71217b5113e@xxx.xxx.xxx.xxx:xxxxx/db

Starting tunnel to mondo on port 10000.
1: none
2: mongo
3: mongodump
4: mongorestore
Which client would you like to start?: 2
Launching 'mongo --host localhost --port 10000 -u u2a13abe8b0aa47b39ade9fd886536a05 -p pa2f356dc4457452abaabd71217b5113e db'

MongoDB shell version: 2.2.2
connecting to: localhost:10000/db

```

**Step 4** Now you can run MOngoDB queries on the services you have created in Cloud Foundry.

```bash
>show collections;
Expense
ExpenseType
system.indexes
system.users
```
