---
title: Grails
description: Grails Application Development with Cloud Foundry
tags:
    - grails
    - groovy
    - mongodb
    - redis
    - code-attached
---

[Grails](http://grails.org) is a framework for rapidly developing web applications that can be deployed to any Java servlet container, such as Tomcat. Based on the dynamic language [Groovy](http://groovy.codehaus.org/) and the [Spring](http://www.springframework.org/) framework, it brings the paradigm of Convention over Configuration to the Java platform with the expressiveness of a Java-like dynamic language.

This guide assumes that you already have Java and [Grails installed](http://grails.org/Installation) and know how to build a simple Grails application. That's all you need to start working with Cloud Foundry.

(If you are working Spring, Roo or Grails and would like to see some code samples, check out this useful link: [https://github.com/SpringSource/cloudfoundry-samples/wiki](https://github.com/SpringSource/cloudfoundry-samples/wiki) )

## Grails Cloud Foundry Plug-in

Let's assume you have a standard Grails application using nothing more than a MySQL database and that you want to deploy this application to Cloud Foundry. The first step is to install the [Cloud Foundry plug-in](http://grails.org/plugin/cloud-foundry). This plug-in adds some commands to Grails that make it trivially easy to deploy and manage your application.

From within your Grails project, execute:

``` bash
$ grails install-plugin cloud-foundry
```

Once the plug-in is installed, you can use the following to see what Cloud Foundry commands are available:

``` bash
$ grails cf-help
```

Most of them are equivalents of the [vmc tool](/tools/vmc/vmc-quick-ref.html) commands.

Next, you need to specify your username and password. The best place to put these is the ~/.grails/settings.groovy file:

``` groovy
    grails.plugin.cloudfoundry.username = "yourusername"
    grails.plugin.cloudfoundry.password = "yourpassword"
```

Don't forget to make this file readable and writable only by you! Now you're ready to deploy your application:

```bash
$ grails prod cf-push
```

This command will first ask you what URL to use. The default, based on the name of the application, is fine as long as the app name is unique on cloudfoundry.com. If it's not, then enter a unique name now rather than accepting the default.

The next question will ask whether you want to create and bind a MySQL service (which you do, so reply with 'y'), and then whether you want to create and bind a PostgreSQL database ('n' for this one, since the application is configured for MySQL). Afterwards, you'll see your application being deployed and started on cloudfoundry.com.

For further information on using the Cloud Foundry plug-in, refer to its [user guide](http://grails-plugins.github.com/grails-cloud-foundry/).

## Services

Cloud Foundry provides a rich set of services, all of which can be used by Grails applications: MySQL, PostgreSQL, MongoDB, Redis, and RabbitMQ. Each of these services has a corresponding Grails plug-in. When these plug-ins are installed, you don't have to configure any connection settings for the associated Cloud Foundry service - it's done all automatically when your application is deployed. And when you first deploy your application with `cf-push`, Grails will ask you whether you want to create and bind the relevant services based on which plug-ins you have installed.

### SQL services

Currently, you can either use MySQL or PostgreSQL on Cloud Foundry if you want a relational database for your application. All you need to access them is the [Hibernate plug-in](http://grails.org/plugin/hibernate), which is included by default in all new Grails applications. You also need to make sure that the relevant driver is available to your application, for example by declaring it in `BuildConfig.groovy`:

```groovy
  grails.project.dependency.resolution = {
      ...
      dependencies {
          runtime "mysql:mysql-connector-java:5.1.18"
          ...
      }
  }

```

and that the JDBC driver class is set in `DataSource.groovy`:

```groovy
  environments {
      production {
          dataSource {
              driverClassName = "com.mysql.jdbc.Driver"
              ...
          }
      }
  }
```

In this case, we're going to deploy the application for the standard production environment, but you could easily set up a 'cloud' or similar environment.
Also, you can easily configure the JDBC URL, username and password for the production environment to point to a local MySQL database because those settings are overridden when the application is deployed to Cloud Foundry.

### Redis

To use the key-value store [Redis](http://redis.io), you need to either install the [Redis plug-in](http://grails.org/plugin/redis) or [Redis for GORM](http://grails.org/plugin/redis-gorm). As with the SQL stores, when you deploy your application to Cloud Foundry it will automatically use whichever Redis service has been bound to it.

The former plug-in provides low-level access to Redis via a `redis` bean, while the latter allows you to map GORM domain classes to Redis. See the [plug-in documentation](http://grails.github.com/inconsequential/redis/manual/index.html) for more information.

### MongoDB

To use the document store [MongoDB](http://www.mongodb.org/), just install the [MongoDB plug-in](http://grails.org/plugin/mongodb).

The MongoDB plug-in for Grails allows you to map GORM entities to MongoDB Collections. It also provides a low-level API for accessing the MongoDB driver for Java directly. Refer to the [plug-in documentation](http://grails.github.com/inconsequential/mongo/manual/index.html) for more information.

## How To

A general tutorial can be found in the blog post ["One-step deployment with Grails and Cloud Foundry"](http://blog.springsource.com/2011/04/12/one-step-deployment-with-grails-and-cloud-foundry/).

In fact, for a super fast start you can try one of our sample applications:

* [Pet Clinic](https://github.com/SpringSource/cloudfoundry-samples/tree/master/petclinic-grails) - Demonstrates how to use the SQL service
* [Grails Twitter](https://github.com/SpringSource/cloudfoundry-samples/tree/master/grailstwitter) - [running here](http://grailstwitter.cloudfoundry.com/) - Demonstrates how to use the MongoDB and Redis services

*WARNING* If you deploy any of these applications yourself, be sure to deploy them with a different application name.

## FAQ

See the [Cloud Foundry FAQ](http://www.cloudfoundry.com/faq) for general questions.

### Can I use the Spring Cloud Namespace from Grails?

Yes you can! See the documentation on [Spring](http://www.springsource.org/documentation) for examples on how to configure additional MySQL, Mongo or Redis instances via the Cloud namespace. This configuration can go into `grails-app/conf/spring/resources.xml` or `grails-app/conf/spring/resources.groovy` using the BeanBuilder DSL.

### How do I get access to Cloud environment variables?

The mechanism is the same for Spring and Grails. See the [Cloud Foundry Environment Variables](https://github.com/SpringSource/cloudfoundry-samples/wiki/Cloud-foundry-environment-variables) page for more information.

### Can I send email from my applications?

Yes, but not via SMTP as you can only send and receive HTTP or HTTPS requests. Instead, you can use an HTTP-based service such as [SendGrid](http://sendgrid.com/), [Amazon SES](http://aws.amazon.com/ses/), or [Mailgun](http://mailgun.net/). See the [support forum](http://support.cloudfoundry.com/entries/20023841) for more info.
