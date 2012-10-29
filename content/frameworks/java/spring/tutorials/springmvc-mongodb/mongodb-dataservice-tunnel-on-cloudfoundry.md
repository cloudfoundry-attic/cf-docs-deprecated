---
title: Accessing Cloud Foundry MongoDB Service
description: Accessing Cloud Foundry MongoDB Service
tags:
    - MongoDB
    - STS
    - vmc
---

## Introduction
This section will teach you how to access the MongoDB service on Cloud Foundry using tunneling.

## Prerequisites
Before you begin this tutorial, you should:

1.  Have installed [Caldecott](/tools/vmc/caldecott.html).

2.  Have installed [STS](http://www.springsource.org/spring-tool-suite-download) or Eclipse.

3.  Have installed [MongoDB](http://www.mongodb.org/downloads).

## Subtopics

+ [Accessing MongoDB on Cloud Foundry via vmc Tunneling](#accessing-mongodb-on-cloud-foundry-via-vmc-tunneling)
+ [Accessing MongoDB on Cloud Foundry via STS](#accessing-mongodb-on-cloud-foundry-via-sts)

## Accessing MongoDB on Cloud Foundry via vmc Tunneling

**Step 1**  Point vmc target to `http://api.cloudfoundry.com` and login to Cloud Foundry using your credentials.

```bash
$ vmc target http://api.cloudfoundry.com
Successfully targeted to [http://api.cloudfoundry.com]

$ vmc login
Attempting login to [http://api.cloudfoundry.com]
Email: **********
Password: **********
Successfully logged into [http://api.cloudfoundry.com]
```

**Step 2** Now enter **vmc services** command to list the services you have created.

```bash
vmc services

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

+------------------+------------+
| Name             | Service    |
+------------------+------------+
| mongodb-f8205    | mongodb    |
| postgres         | postgresql |
| postgresql-806ac | postgresql |
+------------------+------------+


```
**Step 3** If you haven't created MongoDB service already, Create a MongoDB service by using **vmc create-service** command otherwise go to Step 4.

```bash
$ vmc create-service
1: mongodb
2: postgresql
3: mysql
4: rabbitmq
5: redis
Which service would you like to provision?: 1
Creating Service [mongodb-f8205]: OK
```

**Step 4** To get the remote service connection details, enter **vmc tunnel <data service name>**

```bash
vmc tunnel mongodb-f8205
Getting tunnel connection info: OK

Service connection info:
  username : 27b15245-1249-403d-8469-4ac3bb86b28c
  password : 63411578-aeb7-4dea-90b9-cc7941e2520d
  name     : db
  url      : mongodb://27b15245-1249-403d-8469-4ac3bb86b28c:63411578-aeb7-4dea-90b9-cc7941e2520d@172.30.48.63:25171/db

Starting tunnel to mongodb-f8205 on port 10001.
1: none
2: mongo
3: mongodump
4: mongorestore
Which client would you like to start?: 2
Launching 'mongo --host localhost --port 10001 -u 27b15245-1249-403d-8469-4ac3bb86b28c -p 63411578-aeb7-4dea-90b9-cc7941e2520d db'

MongoDB shell version: 2.2.0
connecting to: localhost:10001/db
>

```

**Step 4** Now you can run queries on the services you have created in Cloud Foundry.

## Accessing MongoDB on Cloud Foundry via STS

1. Go to servers, right click to add New Server. Now you can see Cloud Foundry under VMware. Select Cloud Foundry, then click next.
  ![spring-expensereport-login.png](/images/spring_tutorial/cloud_foundry.png)
2. Now enter your Cloud Foundry account information,  choose **VMware Cloud Foundry - https://api.cloudfoundry.com** for the URL, and click **Validate Account**.
  ![cloud foundry account information.png](/images/spring_tutorial/cloud_foundry_account.png)
3. Once the VMware Cloud Foundry server has started, double click it. It will open a new window. Click on the `Applications` tab to see the Services you have created for application.
4. Right click the mongodb service and select **Open Tunnel**.
  ![open-caldecott](/images/spring_tutorial/open_tunnel_mongodb.png)
5. Once complete, STS will give the connection information. Right click to copy all of the details.

    ![open-caldecott](/images/spring_tutorial/caldecott_info_mongodb.png)

6. Now go to terminal and connect to the mongodb client using the tunneling information.

```bash
mongo --host localhost --port 10001 -u 27b15245-1249-403d-8469-4ac3bb86b28c -p 63411578-aeb7-4dea-90b9-cc7941e2520d
```
Now you can run queries on the services you have created in Cloud Foundry.
