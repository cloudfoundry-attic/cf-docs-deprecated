---
title: Accessing Cloud Foundry Services
description: Accessing Cloud Foundry Services
tags:
    - PostgreSQL
    - STS
    - vmc
---

## Introduction
This section will teach you how to access the PostgreSQL service on Cloud Foundry using tunneling.

## Prerequisites
Before you begin this tutorial, you should:

1.  Have deployed Spring Application with PostgreSQL service on Cloud Foundry.

2.  Have installed [Caldecott](/tools/vmc/caldecott.html) application.

3.  Have installed [STS](/tools/vmc/caldecott.html) or Eclipse IDE.

4.  Have installed [PostgreSQL](http://www.postgresql.org/download/).

## PostgreSQL Service accessing on Cloud Foundry through vmc tunneling

**Step 1**.  Point vmc target to `http://api.cloudfoundry.cpm` and login to Cloud Foundry using your credentials.

```bash
$ vmc target http://api.cloudfoundry.com
Successfully targeted to [http://api.cloudfoundry.com]

$ vmc login
Attempting login to [http://api.cloudfoundry.com]
Email: senthil.s@imaginea.com
Password: **********
Successfully logged into [http://api.cloudfoundry.com]
```

**Step 2** Now give **vmc services** command it will list services created by you.

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
| postgres | postgresql |
+----------+------------+

```

**Step 3** To get the remote service connection details, give command **vmc tunnel <data service name>**

```bash
$ vmc tunnel postgres
Getting tunnel connection info: OK

Service connection info: 
  username : u2a13abe8b0aa47b39ade9fd886536a05
  password : pa2f356dc4457452abaabd71217b5113e
  name     : da5857027ab844d7d9f474ceaf196c9d0

Starting tunnel to postgres on port 10000.
1: none
2: psql
Which client would you like to start?: 2
Launching 'psql -h localhost -p 10000 -d da5857027ab844d7d9f474ceaf196c9d0 -U u2a13abe8b0aa47b39ade9fd886536a05 -w'

psql (9.1.5, server 9.0.4)
WARNING: psql version 9.1, server version 9.0.
         Some psql features might not work.
Type "help" for help.

da5857027ab844d7d9f474ceaf196c9d0=> 

```

**Step 4** Now you can run queries on the services you have created in Cloud Foundry.

```bash
da5857027ab844d7d9f474ceaf196c9d0=> \dt
                         List of relations
 Schema |     Name      | Type  |               Owner               
--------+---------------+-------+-----------------------------------
 public | attachment    | table | u4d59d15759c4460285dc892b72ef6fcb
 public | expense       | table | u4d59d15759c4460285dc892b72ef6fcb
 public | expensereport | table | u4d59d15759c4460285dc892b72ef6fcb
 public | expensetype   | table | u4d59d15759c4460285dc892b72ef6fcb
 public | role          | table | u4d59d15759c4460285dc892b72ef6fcb
 public | users         | table | u4d59d15759c4460285dc892b72ef6fcb
(6 rows)
```

## PostgreSQL Service accessing on Cloud Foundry through STS

1.  Go to servers view, right click add New Server. Now you can see Cloud Foundry under VMware. Select Cloud Foundry,then click next.It will ask your account information and Choose **VMware Cloud Foundry - http://api.cloudfoundry.com** from the URL.

	![spring-expensereport-login.png](/images/spring_tutorial/cloud_foundry.png)

     Fig 5.1

2.  Now give your Cloud Foundry account information. **Click validate** and make sure you have entered  valid credentials.

	![cloud foundry account information.png](/images/spring_tutorial/cloud_foundry_account.png)

     Fig 5.2

3.  Once Vmware Cloud Foundry server started, double click it. It will open the new window and select `Applications` tab where you can see Services that you have created for application.

4.  Select postgresql service and right click it, now select open caldecott.

	![open-caldecott](/images/spring_tutorial/open_tunnel.png)

     Fig 5.3

5.  Once the tunnel starts completed STS will give the connection information copy the details.

	![open-caldecott](/images/spring_tutorial/caldecott_info.png)

     Fig 5.4

6.  Open Database development perspective and add New connection, select PostgreSQL connection profile and click next, it will open window shown below and enter the Cloud Foundry service connection information. Click test connection to ensure you have entered correct details.

	![spring-expensereport-login.png](/images/spring_tutorial/open_service_in_local.png)

     Fig 5.5

	![spring-expensereport-login.png](/images/spring_tutorial/enter_cloud_foundry_service_auth_details.png)

     Fig 5.6

7.  Once ping succeeds click next it will create a new connection. Right click the Cloud Foundry service connection and select open Sql Scrapbook, it will open a new Sql Scrpbook, now select your Cloud Foundry service connection.

	![open-caldecott](/images/spring_tutorial/ping_succeed.png)

     Fig 5.7

	![open-caldecott](/images/spring_tutorial/sql-scrapbook.png)

     Fig 5.8

8.  Now you can run queries on the services you have created in Cloud Foundry.
