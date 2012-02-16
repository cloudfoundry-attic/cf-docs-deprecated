---
title: Using MySQL with Grails
description: Grails Development with the MySQL Cloud Foundry Service
tags:
    - groovy
    - mysql
---

Currently, you can either use MySQL or PostgreSQL on Cloud Foundry if you want a relational database for your application.
All you need to access them is the [Hibernate plug-in](http://grails.org/plugin/hibernate), which is included by default
in all new Grails applications. You also need to make sure that the relevant driver is available to your application,
for example by declaring it in `BuildConfig.groovy`:

``` groovy
grails.project.dependency.resolution = {
    ...
    dependencies {
        runtime "mysql:mysql-connector-java:5.1.18"
        ...
    }
}
```

and that the JDBC driver class is set in `DataSource.groovy`:

``` groovy
environments {
    production {
        dataSource {
            driverClassName = "com.mysql.jdbc.Driver"
            ...
        }
    }
}
```

In this case, we're going to deploy the application for the standard production environment, but you could easily set
up a 'cloud' or similar environment. Also, you can easily configure the JDBC URL, username and password for the production
environment to point to a local MySQL database because those settings are overridden when the application is deployed to Cloud Foundry.
