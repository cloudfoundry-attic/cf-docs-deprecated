---
title: Accessing Cloud Foundry Services
description: Accessing Cloud Foundry Services
tags:
    - vmc
    - redis
---

## Introduction
This section will teach you how to access the Redis service on Cloud Foundry using tunneling.

## Prerequisites
Before you begin this tutorial, you should:

1. Have installed [Caldecott](/tools/vmc/caldecott.html).
2. Have installed [STS](http://www.springsource.org/spring-tool-suite-download) or Eclipse.
3. Have installed [Redis](http://www.redis.io/).

## Accessing Redis on Cloud Foundry via vmc Tunneling

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
| redis1   | redis      |
+----------+------------+

```

**Step 3** To get the remote service connection details, enter **vmc tunnel data service name**

```bash
$ vmc tunnel redis1
Getting tunnel connection info: OK

Service connection info:
  password : u2a13abe8b0aa47b39ade9fd886536a05

Starting tunnel to redis1 on port 10000.
1: none
2: redis-cli
Which client would you like to start?: 2
Launching 'redis-cli -h localhost -p 10000 -a u2a13abe8b0aa47b39ade9fd886536a05'

redis localhost:10000>
```

**Step 4** Now you can run Redis commands on the services you have created in Cloud Foundry.

```bash
redis localhost:10000> keys *
1) "expenseTypeCount"
2) "\xac\xed\x00\x05t\x00\rexpenseType:1"
3) "\xac\xed\x00\x05t\x00\rexpenseType:2"
4) "\xac\xed\x00\x05t\x00\rexpenseType:3"
5) "\xac\xed\x00\x05t\x00\rexpenseType:4"
6) "expenseCount"
(2.01s)
redis localhost:10000>
```
