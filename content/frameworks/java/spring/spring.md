---
title: Spring
description: Spring Application Development with Cloud Foundry
tags:
    - spring
    - mongodb
    - redis
    - code-attached
---

This section provides practical information for Spring developers who are using Cloud Foundry to deploy their applications.  In particular, the section describes Spring programming and packaging topics that are specific to the Cloud Foundry environment, and what you need to do to use the provided services, such as MySQL and RabbitMQ.

(If you're interested in using Spring Insight to monitor your Java applications on Cloud Foundry, read [this](/frameworks/java/spring/spring-insight.html).)

(If you are working Spring, Roo or Grails and would like to see some code samples, check out this useful link: [https://github.com/SpringSource/cloudfoundry-samples/wiki](https://github.com/SpringSource/cloudfoundry-samples/wiki) )

## Prerequisites

It is assumed that you have already installed either VMC, SpringSource STS, or Eclipse and that you have used one of these tools to deploy a simple HelloWorld application to Cloud Foundry, either to your local Micro version or to the service hosted at cloudfoundry.com.  It is also assumed that you are already a proficient Spring application developer.  For additional information, see:

+ [Installing the Command-Line Utility (VMC)](/tools/vmc/installing-vmc.html)
+ [Installing the Cloud Foundry Integration Extension for SpringSource Tool Suite (STS) and Eclipse](/tools/STS/configuring-STS.html)
+ [Getting Started With Spring](http://www.springsource.org/get-started)

## Subtopics

+ [Programming and Packaging Spring Applications](#programming-and-packaging-spring-applications)
+ [Using Cloud Foundry Services In Spring Applications](#using-cloud-foundry-services-in-spring-applications)
+ [RabbitMQ And Spring: Additional Programming Information](#rabbitmq-and-spring-additional-programming-information)
+ [Using Spring Profiles to Conditionalize Cloud Foundry Configuration](#using-spring-profiles-to-conditionalize-cloud-foundry-configuration)
+ [Sending Email From Spring Applications Deployed to Cloud Foundry](#sending-email-from-spring-applications-deployed-to-cloud-foundry)
+ [Accessing Cloud Foundry Properties](#accessing-cloud-foundry-properties)

## Programming and Packaging Spring Applications

In general, you do not have to do anything "special" when programming or packaging Spring applications that you want to deploy to Cloud Foundry.  In other words, if your Spring application runs on tc Server or Apache Tomcat, then it will also run on Cloud Foundry without any changes.

It is usually best to package your Spring application in a `*.war` file so that `vmc push` will automatically detect that it is a Spring application, but you can also deploy it as an exploded directory if you want.

There are, however, a few things about the Cloud Foundry environment that you should be aware of that might affect how your deployed application runs:

+  Local disk storage is ephemeral.  In other words, local disk storage is not guaranteed to persist for the entire life of the application.  This is because Cloud Foundry creates a new virtual disk every time you restart your application.  Additionally, Cloud Foundry restarts all applications after it updates its own environment.  This means that, although your application is *able* to write local files while it is running, the files will disappear after the application restarts.   If the files your application is writing are temporary, then this should not be a problem.  However, if your application needs the data in the files to persist, then you must use one of the data services to manage the data.  In this scenario, MongoDB is a good choice because it is a document-oriented database.

+ Cloud Foundry uses Apache Tomcat as its application server, and it runs your application within the `root` context.   This is different from normal Apache Tomcat in which the context path is determined by the name of the `*.war` file in which the application is packaged.

+ HTTP sessions are not replicated, but HTTP traffic is sticky.  This means that if a your application crashes or restarts, the HTTP sessions are lost.

+ External users can access your application only via the URL provided by the `vmc push` command (or equivalent STS command).  Although your application might be able to internally listen to other ports (such as the JMX port for the MBean server), external users of your application will not be able to listen on these ports.

## Using Cloud Foundry Services In Spring Applications

If your Spring application requires services (such as a database or RabbitMQ), you might be able to deploy your application to Cloud Foundry *without changing a single line of code*.  In this case, Cloud Foundry automatically re-configures the relevant bean definitions to bind them to cloud services.  See [Determining Whether Your Application Can Be Auto-Configured](#determining-whether-your-application-can-be-auto-configured) for details.

If your Spring application cannot take advantage of Cloud Foundry's auto-reconfiguration feature, or you want more control over the configuration, the additional steps you must take are very simple and easy.  See [Explicitly Configuring Your Application to Use Cloud Foundry Services](#explicitly-configuring-your-application-to-use-cloud-foundry-services).

## Determining Whether Your Application Can Be Auto-Configured

You will likely be able to deploy many of your existing Spring applications to Cloud Foundry without changing a single line of code, even in the case that your application requires a service such as a relational database or RabbitMQ. This is because Cloud Foundry automatically detects the type of service your application needs, and if its configuration falls within a small [set of limitations](#limitations-of-auto-reconfiguration), Cloud Foundry will automatically reconfigure it so that it binds to service instances that Cloud Foundry creates and maintains itself.

With auto-reconfiguration, Cloud Foundry creates the database or connection factory itself, using its own values for properties such as host, port, username and so on.  For example, if you have a single `javax.sql.DataSource` bean in your application context that Cloud Foundry reconfigures and binds to its own database service, Cloud Foundry doesn't use the username and password and driver URL you originally specified.  Rather, it uses its own internal values.  This is transparent to the application, which really only cares about having a relational database to which it can write data but doesn't really care what the specific properties are that created the database.

The following sections describe, for each supported service, the type of bean that Cloud Foundry detects if auto-configuration occurs.

## Relational Database (MySQL and vFabric Postgres)

Auto-reconfiguration occurs if Cloud Foundry detects a `javax.sql.DataSource` bean.   The following snippet of a Spring application context file shows an example of defining this type of bean which Cloud Foundry will in turn detect and potentially auto-reconfigure:

```xml
<bean class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close" id="dataSource">
    <property name="driverClassName" value="org.h2.Driver" />
    <property name="url" value="jdbc:h2:mem:" />
    <property name="username" value="sa" />
    <property name="password" value="" />
</bean>
```

The relational database that Cloud Foundry actually uses depends on the service instance you explicitly bind to your application when you deploy it: MySQL or vFabric Postgres.  Cloud Foundry creates either a commons DBCP or Tomcat datasource.

Cloud Foundry will internally generate values for the following properties:  driverClassName, url, username, password, validationQuery.

## MongoDB

You must be using [Spring Data MongoDB](http://www.springsource.org/spring-data/mongodb) 1.0 M4 or later for auto-reconfiguration to work.

Auto-reconfiguration occurs if Cloud Foundry detects a `org.springframework.data.document.mongodb.MongoDbFactory` bean.   The following snippet of a Spring application context file shows an example of defining this type of bean which Cloud Foundry will in turn detect and potentially auto-reconfigure:

```xml
<mongo:db-factory
    id="mongoDbFactory"
    dbname="pwdtest"
    host="127.0.0.1"
    port="1234"
    username="test_user"
    password="test_pass"  />
```

Cloud Foundry will create a `SimpleMOngoDbFactory` with its own values for the following properties:  host, port, username, password, dbname.

## Redis

You must be using [Spring Data Redis](http://www.springsource.org/spring-data/redis) 1.0 M4 or later for auto-reconfiguration to work.

Auto-reconfiguration occurs if Cloud Foundry detects a `org.springframework.data.redis.connection.RedisConnectionFactory` bean.   The following snippet of a Spring application context file shows an example of defining this type of bean which Cloud Foundry will in turn detect and potentially auto-reconfigure:

```xml
<bean id="redis"
      class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory"
      p:hostName="localhost" p:port="6379"  />
```
Cloud Foundry will create a `JedisConnectionFactory` with its own values for the following properties:  host, port, password.   This means that you must package the Jedis JAR in your application.  Cloud Foundry does not currently support the JRedis and RJC implementations.

## RabbitMQ

You must be using [Spring AMQP](http://www.springsource.org/spring-amqp) 1.0 or later for auto-reconfiguration to work.  Spring AMQP provides publishing, multi-threaded consumer generation, and message converters.  It also facilitates management of AMQP resources while promoting dependency injection and declarative configuration.

Auto-reconfiguration occurs if Cloud Foundry detects a `org.springframework.amqp.rabbit.connection.ConnectionFactory` bean.   The following snippet of a Spring application context file shows an example of defining this type of bean which Cloud Foundry will in turn detect and potentially auto-reconfigure:

```xml
<rabbit:connection-factory
    id="rabbitConnectionFactory"
    host="localhost"
    password="testpwd"
    port="1238"
    username="testuser"
    virtual-host="virthost" />
```

Cloud Foundry will create a `org.springframework.amqp.rabbit.connection.CachingConnectionFactory` with its own values for the following properties:  host, virtual-host, port, username, password.

## Limitations of Auto-Reconfiguration

Cloud Foundry auto-reconfigures applications only if the following items are true for your application:

+ You bind only *one* service instance of a given service type to your application.  In this context, MySQL and vFabric Postgres are considered the same service type (relational database), so if you have bound both a MySQL and a vFabric Postgres service to your application, auto-reconfiguration will *not* occur.
+ You include only *one* bean of a matching type in your Spring application context file.  For example, you can have only one bean of type `javax.sql.DataSource`.

Also note that if auto-reconfiguration occurs, but you have customized the configuration of the service (such as the pool size or connection properties), Cloud Foundry ignores the customizations.

## Opting Out of Auto-Reconfiguration

Sometimes you may not want Cloud Foundry to auto-reconfigure your Spring application in the ways described in this section.  There are two ways to opt out:

+ When you deploy your application using VMC or STS, specify that the framework is `JavaWeb` instead of `Spring`.  Note that in this case your application will not be able to take advantage of the Spring Profiles feature.
+ Use the `<cloud:>` namespace elements in your Spring application context file to explicitly create a bean that represents a service.  This makes auto-reconfiguration unnecessary.  See [Explicitly Configuring Your Application to Use Cloud Foundry Services](#explicitly-configuring-your-application-to-use-cloud-foundry-services).

## Explicitly Configuring Your Application to Use Cloud Foundry Services

This section describes the simple configuration steps you must perform in your Spring applications so they can use the Cloud Foundry services.  See [Configuring Applications to Use Cloud Foundry](/frameworks.html) for the complete list of supported Cloud Foundry services.

The easiest way to use Cloud Foundry services in your Spring applications is to declare the `<cloud:>` namespace, point it to the Cloud Foundry Schema, then use the service-specific elements defined in the `<cloud>` namespace.  For example, with just a single line of XML in your application context file you can create a JDBC data source or a RabbitMQ connection factory that you can use in your specific bean definitions.  You can configure multiple data sources or connection factories if your application requires it.   You can also further configure these cloud services if you want, although it is completely optional because Cloud Foundry uses common-place configuration values to create typical service instances that are adequate for most uses.  In sum, using the `<cloud:>` namespace provides you with as much control as you want over the number and type of Cloud Foundry services that your application uses.

The basic steps to update your Spring application to use any of the Cloud Foundry services are as follows:

* Update your application build process to include a dependency on the `org.cloudfoundry.cloudfoundry-runtime` artifact.  For example, if you use Maven to build your application, the following `pom.xml` snippet shows how to add this dependency:

```xml
<dependencies>
    <dependency>
        <groupId>org.cloudfoundry</groupId>
        <artifactId>cloudfoundry-runtime</artifactId>
        <version>0.8.1</version>
    </dependency>

    <!-- additional dependency declarations -->
</dependencies>
```

*  Update your application build process to include the Spring Framework Milestone repository.  The following `pom.xml` snippet shows how to do this in Maven:

```xml
<repositories>
    <repository>
          <id>org.springframework.maven.milestone</id>
           <name>Spring Maven Milestone Repository</name>
           <url>http://maven.springframework.org/milestone</url>
           <snapshots>
                   <enabled>false</enabled>
           </snapshots>
    </repository>

    <!-- additional repository declarations -->
</repositories>
```

*  In your Spring application, update all application context files that will include the Cloud Foundry service declarations, such as a data source, by adding the `<cloud:>` namespace declaration and the location of the Cloud Foundry services Schema, as shown in the following snippet:

```xml

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:cloud="http://schema.cloudfoundry.org/spring"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context-3.1.xsd
        http://schema.cloudfoundry.org/spring
        http://schema.cloudfoundry.org/spring/cloudfoundry-spring.xsd
        >

    <!-- bean declarations -->

</beans>
```

*  You can now specify the Cloud Foundry services in the Spring application context file by using the `<cloud:>` namespace along with the name of specific elements, such as `data-source`.  Cloud Foundry provides elements for each of the supported services:  database (MySQL and vFabric Postgres), Redis, MongoDB, and RabbitMQ.

    The following example shows a simple data source configuration that will be injected into a JdbcTemplate using the `<cloud:data-source>` element.

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:cloud="http://schema.cloudfoundry.org/spring"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context-3.1.xsd
        http://schema.cloudfoundry.org/spring
        http://schema.cloudfoundry.org/spring/cloudfoundry-spring.xsd
        >

    <cloud:data-source id="dataSource" />

    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
      <property name="dataSource" ref="dataSource" />
    </bean>

        <!-- additional beans in your application -->

</beans>
```

When you later deploy the application using VMC or STS, you bind a specific data service (such as MySQL or vFabric Postgres) to it and Cloud Foundry creates an instance of the service.  Note that in the example above, you did not specify typical data source properties such as `driverClassName` or `url` or `username` - this is because Cloud Foundry automatically takes care of those properties for you.

For complete information about all the `<cloud:>` elements you can use in your Spring application context file to access Cloud Foundry services, see the following sections:

+ [\<cloud:data-source\>: Configure a JDBC Data Source for Either MySQL or vFabric Postgres Databases](#clouddata-source)
+ [\<cloud:mongo-db-factory\>: Configure a MongoDB Connection Factory](#cloudmongo-db-factory)
+ [\<cloud:redis-connection-factory\>: Configure a Redis Connection Factory](#cloudredis-connection-factory)
+ [\<cloud:rabbit-connection-factory\>: Configure a RabbitMQ Connection Factory](#cloudrabbit-connection-factory)
+ [\<cloud:service-scan\> Injecting Services Into @Autowired Beans ](#cloudservice-scan)
+ [\<cloud:properties\> Get Cloud Foundry Service Information](#cloudproperties)

* After you have finished specifying all the Cloud Foundry services you are going to use in your application, you use the standard Cloud Foundry client commands (VMC, SpringSource Tool Suite, or the Eclipse plugin) to create instances of these services, bind them to your applications, then deploy your applications to the hosted Cloud Foundry (cloudfoundry.com) or to your local Micro Cloud Foundry instance.  See [Deploying Applications](/tools/deploying-apps.html) for details on how to use these tools.

## \<cloud:data-source\>

The `<cloud:data-source>` element provides an easy way for you to configure a JDBC data source for your Spring application.    Later, when you actually deploy the application, you bind a particular database service instance to it, such as MySQL or vFabric Postgres.

The following example shows a simple way to configure a JDBC data source that will be injected into a `org.springframework.jdbc.core.JdbcTemplate` bean:

```xml
<cloud:data-source id="dataSource" />

<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
  <property name="dataSource" ref="dataSource" />
</bean>
```
In the preceding example, note that no specific information about the datasource is supplied, such as the JDBC driver classname, the specific URL to access the database, and the database users.  Instead, Cloud Foundry will take care of all of that at runtime, using appropriate information from the specific type of database service instance you bind to your application.

**Attributes**

The following table lists the attributes of the `<cloud:data-source>` element.

<table class="std">
 <tr>
   <th>Attribute</th>
   <th>Description</th>
   <th>Type</th>
 </tr>
 <tr>
   <td>id</td>
   <td>The ID of this data source.  The JdbcTemplate bean uses this ID when it references the data source. <br>Default value is the name of the bound service instance.</td>
   <td>String</td>
 </tr>
 <tr>
   <td>service-name</td>
   <td>The name of the data source service. <br>You specify this attribute only if you are binding multiple database services to your application and you want to specify which particular service instance binds to a particular Spring bean.  The default value is the name of the bound service instance.</td>
   <td>String</td>
 </tr>
</table>

**Advanced Data Source Configuration**

The section above showed how to configure a very simple JDBC data source; Cloud Foundry uses the most common configuration options when it actually creates the data source at runtime.  You can, however, specify some of these configuration options using the following two child elements of `<cloud:data-source>`:  `<cloud:connection>` and `<cloud:pool>`.

The `<cloud:connection>` child element takes a single String attribute (`properties`) that you use to specify connection properties you want to send to the JDBC driver when establishing new database connections.  The format of the string must be semi-colon separated name/value pairs  (`[propertyName=property;]`).

The `<cloud:pool>` child element takes the following two attributes:

<table class="std">
 <tr>
   <th>Attribute</th>
   <th>Description</th>
   <th>Type</th>
   <th>Default</th>
 </tr>
 <tr>
   <td>pool-size</td>
   <td>Specifies the size of the connection pool.  Set the value to either the maximum number of connections in the pool, or a range of the minimum and maximum number of connections separated by a dash.</td>
   <td>int</td>
   <td>Default minimum is 0.  Default maximum is 8. These are the same defaults as the Apache Commons Pool.</td>
 </tr>
 <tr>
   <td>max-wait-time</td>
   <td>In the event that there are no available connections, this attribute specifies the maximum number of milliseconds that the connection pool waits for a connection to be returned before throwing an exception. Specify `-1` to indicate that the connection pool should wait forever. </td>
   <td>int</td>
   <td>Default value is `-1` (forever).</td>
 </tr>
</table>

The following example shows how to use these advanced data source configuration options:

```xml
        <cloud:data-source id="mydatasource">
            <cloud:connection properties="charset=utf-8" />
            <cloud:pool pool-size="5-10" max-wait-time="2000" />
        </cloud:data-source>
```
In the preceding example, the JDBC driver is passed the property that specifies that it should use the UTF-8 character set.  The minimum and maximum number of connections in the pool at any given time is 5 and 10, respectively.  The maximum amount of time that the connection pool waits for a returned connection if there are none available is 2000 milliseconds (2 seconds), after which the JDBC connection pool throws an exception.

## \<cloud:mongo-db-factory \>

The `<cloud:mongo-db-factory>` provides a simple way for you to configure a MongoDB connection factory for your Spring application.

The following example shows a MongoDbFactory configuration that will be injected into a `org.springframework.data.mongodb.core.MongoTemplate` object:

```xml
<cloud:mongo-db-factory id="mongoDbFactory" />

<bean id="mongoTemplate" class="org.springframework.data.mongodb.core.MongoTemplate">
    <constructor-arg ref="mongoDbFactory"/>
</bean>
```

**Attributes**

The following table lists the attributes of the `<cloud:mongo-db-factory>` element.

<table class="std">
 <tr>
   <th>Attribute</th>
   <th>Description</th>
   <th>Type</th>
 </tr>
 <tr>
   <td>id</td>
   <td>The ID of this MongoDB connection factory.  The MongoTemplate bean uses this ID when it references the connection factory. <br>Default value is the name of the bound service instance.</td>
   <td>String</td>
 </tr>
 <tr>
   <td>service-name</td>
   <td>The name of the MongoDB service. <br>You specify this attribute only if you are binding multiple MongoDB services to your application and you want to specify which particular service instance binds to a particular Spring bean.  The default value is the name of the bound service instance.</td>
   <td>String</td>
 </tr>
 <tr>
   <td>write-concern</td>
   <td>Controls the behavior of writes to the data store.  The values of this attribute correspond to the values of the `com.mongodb.WriteConcern` class.
   <p>If you do not specify this attribute, then no `WriteConcern` is set for the database connections and all writes default to NORMAL.  </p>
   <p>The possible values for this attribute are as follows:</p>
	<ul>
	  <li><b>NONE</b>: No exceptions are raised, even for network issues.</li>
	  <li><b>NORMAL</b>: Exceptions are raised for network issues, but not server errors.</li>
	  <li><b>SAFE</b>: MongoDB service waits on a server before performing a write operation.  Exceptions are raised for both network and server errors.</li>
	  <li><b>FSYNC_SAVE</b>: MongoDB service waits for the server to flush the data to disk before performing a write operation.  Exceptions are raised for both network and server errors.</li>
	</ul>
    </td>
   <td>String</td>
 </tr>
</table>

## Advanced MongoDB Configuration

The preceding section shows how to configure a simple MongoDB connection factory using default values for the options.   This is adequate for many environments.  However, you can further configure the connection factory by specifying the optional `<cloud:mongo-options>` child element of `<cloud:mongo-db-factory>`.

The `<cloud:mongo-options>` child element takes the following two attributes:

<table class="std">
 <tr>
   <th>Attribute</th>
   <th>Description</th>
   <th>Type</th>
   <th>Default</th>
 </tr>
 <tr>
   <td>connections-per-host</td>
   <td>Specifies the maximum number of connections allowed per host for the MongoDB instance. Those connections will be kept in a pool when idle. Once the pool is exhausted, any operation requiring a connection will block while waiting for an available connection.</td>
   <td>int</td>
   <td>10</td>
 </tr>
 <tr>
   <td>max-wait-time</td>
   <td>Specifies the maximum wait time (in milliseconds) that a thread waits for a connection to become available.</td>
   <td>int</td>
   <td>120,000 (2 minutes)</td>
 </tr>
</table>

The following example shows how to use the advanced MongoDB options:

```xml
    <cloud:mongo-db-factory id="mongoDbFactory" write-concern="FSYNC_SAFE">
        <cloud:mongo-options connections-per-host="12" max-wait-time="2000" />
    </cloud:mongo-db-factory>
```

In the preceding example, the maximum number of connections is set to 12 and the maximum amount of time that a thread waits for a connection is 1 second.  The WriteConcern is also specified to be the safest possible (`FSYNC_SAFE`).

## \<cloud:redis-connection-factory \>

The `<cloud:redis-connection-factory>` provides a simple way for you to configure a Redis connection factory for your Spring application.

The following example shows a `RedisConnectionFactory` configuration that will be injected into a `org.springframework.data.redis.core.StringRedisTemplate` object:

``` xml
    <cloud:redis-connection-factory id="redisConnectionFactory" />

    <bean id="redisTemplate" class="org.springframework.data.redis.core.StringRedisTemplate">
        <property name="connection-factory" ref="redisConnectionFactory"/>
    </bean>
```

**Attributes**

The following table lists the attributes of the `<cloud:redis-connection-factory>` element.

<table class="std">
 <tr>
   <th>Attribute</th>
   <th>Description</th>
   <th>Type</th>
 </tr>
 <tr>
   <td>id</td>
   <td>The ID of this Redis connection factory.  The RedisTemplate bean uses this ID when it references the connection factory. <br>Default value is the name of the bound service instance.</td>
   <td>String</td>
 </tr>
 <tr>
   <td>service-name</td>
   <td>The name of the Redis service. <br>You specify this attribute only if you are binding multiple Redis services to your application and you want to specify which particular service instance binds to a particular Spring bean.  The default value is the name of the bound service instance.</td>
   <td>String</td>
 </tr>
</table>

**Advanced Redis Configuration**

The preceding section shows how to configure a very simple Redis connection factory; Cloud Foundry uses the most common configuration options when it actually creates the factory at runtime.  You can, however, change some of these configuration options using the`<cloud:pool>` child element of `<cloud:redis-connection-factory>`.

The `<cloud:pool>` child element takes the following two attributes:

<table class="std">
 <tr>
   <th>Attribute</th>
   <th>Description</th>
   <th>Type</th>
   <th>Default</th>
 </tr>
 <tr>
   <td>pool-size</td>
   <td>Specifies the size of the connection pool.  Set the value to either the maximum number of connections in the pool, or a range of the minimum and maximum number of connections separated by a dash.</td>
   <td>int</td>
   <td>Default minimum is 0.  Default maximum is 8. These are the same defaults as the Apache Commons Pool.</td>
 </tr>
 <tr>
   <td>max-wait-time</td>
   <td>In the event that there are no available connections, this attribute specifies the maximum number of milliseconds that the connection pool waits for a connection to be returned before throwing an exception. Specify `-1` to indicate that the connection pool should wait forever. </td>
   <td>int</td>
   <td>Default value is `-1` (forever).</td>
 </tr>
</table>

The following example shows how to use these advanced Redis configuration options:

``` xml
    <cloud:redis-connection-factory id="myRedisConnectionFactory">
        <cloud:pool pool-size="5-10" max-wait-time="2000" />
    </cloud:redis-connection-factory>
```

In the preceding example, the minimum and maximum number of connections in the pool at any given time is 5 and 10, respectively.  The maximum amount of time that the connection pool waits for a returned connection if there are none available is 2000 milliseconds (2 seconds), after which the Redis connection pool throws an exception.

## \<cloud:rabbit-connection-factory \>

The `<cloud:rabbit-connection-factory>` provides a simple way for you to configure a RabbitMQ connection factory for your Spring application.

The following complete example of a Spring application contenxt file shows a `RabbitConnectionFactory` configuration that will be injected into a `rabbitTemplate` object.  The example also uses the `<rabbit:>` namespace to perform RabbitMQ-specific configurations, as explained after the example:

``` xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:rabbit="http://www.springframework.org/schema/rabbit"
       xmlns:cloud="http://schema.cloudfoundry.org/spring"
       xsi:schemaLocation="http://www.springframework.org/schema/mvc   http://www.springframework.org/schema/mvc/spring-mvc-3.0.xsd
           http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.0.xsd
           http://www.springframework.org/schema/rabbit http://www.springframework.org/schema/rabbit/spring-rabbit-1.0.xsd
           http://schema.cloudfoundry.org/spring http://schema.cloudfoundry.org/spring/cloudfoundry-spring.xsd">

    <!-- Obtain a connection to the RabbitMQ via cloudfoundry-runtime: -->
    <cloud:rabbit-connection-factory id="rabbitConnectionFactory" />

    <!-- Set up the AmqpTemplate/RabbitTemplate: -->
    <rabbit:template id="rabbitTemplate"
        connection-factory="rabbitConnectionFactory" />

    <!-- Request that queues, exchanges and bindings be automatically declared on the broker: -->
    <rabbit:admin connection-factory="rabbitConnectionFactory"/>

    <!-- Declare the "messages" queue: -->
    <rabbit:queue name="messages" durable="true"/>

    <!-- additional beans in your application -->

</beans>
```

In the preceding example, note the definition and location of the `<rabbit:>` namespace at the top of the XML file.  This namespace is then used to configure RabbitTemplate and RabbitAdmin objects as the main entry points to Spring AMQP and to declare a queue called `messages` within the RabbitMQ broker.

See [RabbitMQ And Spring: Additional Programming Information](#rabbitmq-and-spring-additional-programming-information) for additional information about using RabbitMQ in your Spring applications.

**Attributes**

The following table lists the attributes of the `<cloud:rabbit-connection-factory>` element.

<table class="std">
 <tr>
   <th>Attribute</th>
   <th>Description</th>
   <th>Type</th>
 </tr>
 <tr>
   <td>id</td>
   <td>The ID of this RabbitMQ connection factory.  The RabbitTempalte bean uses this ID when it references the connection factory. <br>Default value is the name of the bound service instance.</td>
   <td>String</td>
 </tr>
 <tr>
   <td>service-name</td>
   <td>The name of the RabbitMQ service. <br>You specify this attribute only if you are binding multiple RabbitMQ services to your application and you want to specify which particular service instance binds to a particular Spring bean.  The default value is the name of the bound service instance.</td>
   <td>String</td>
 </tr>
</table>

**Advanced RabbitMQ Configuration**

The preceding section shows how to configure a very simple RabbitMQ connection factory; Cloud Foundry uses the most common configuration options when it actually creates the factory at runtime.  You can, however, change some of these configuration options using the`<cloud:rabbit-options>` child element of `<cloud:rabbit-connection-factory>`.

The `<cloud:rabbit-options>` child element defines one attribute called `channel-cache-size` which you can set to specify the size of the channel cache size.  The default value is 1.

The following example shows how to use these advanced RabbitMQ configuration options:

``` xml
        <cloud:rabbit-connection-factory id="rabbitConnectionFactory" >
            <cloud:rabbit-options channel-cache-size="10" />
        </cloud:rabbit-connection-factory>
```

In the preceding example, the channel cache size of the RabbitMQ connection factory is set to 10.

## \<cloud:service-scan\>

The `<cloud:service-scan>` element scans all services bound to the application and creates a Spring bean of an appropriate type for each one that has been annoted with the `@org.springframework.beans.factory.annotation.Autowired` annotation.  The `<cloud:service-scan>` element acts as a cloud equivalent of `<context:component-scan>` in core Spring, which scans the CLASSPATH for beans with certain annotations and creates a bean for each.

The `<cloud:service-scan>` is especially useful during the initial phases of application development, because you can get immediate access to service beans without explicitly adding a `<cloud:>` element to your Spring application context file for each new service that you bind.

The `<cloud:service-scan>` element has no attributes or child elements; for example:

``` xml
         <cloud:service-scan />
```

In your Java code, you must annotate each dependency with `@Autowired` so that a bean of the corresponding service is automatically created.  For example:

``` java
  package cf.examples;

  import org.springframework.beans.factory.annotation.Autowired;

  ....

  @Autowired DataSource dataSource;
  @Autowired ConnectionFactory rabbitConnectionFactory;
  @Autowired RedisConnectionFactory redisConnectionFactory;
  @Autowired MongoDbFactory mongoDbFactory;

  ...
```

Use of only the `@Autowired` annotation is adequate if you have bound only *one* service of each service type to your application.  If you bind more than one (for example, you bind two different MySQL service instances to the same application) then you must also use the `@Qualifier` annotation to match the Spring bean with the specific service instance.

For example, assume you bound two MySQL services (named `inventory-db` and `pricing-db`) to your application; use the `@Qualifier` annotation as shown in the following example to specify which service instance applies to which Spring bean:

``` java
  @Autowired @Qualifier("inventory-db") DataSource inventoryDataSource;
  @Autowired @Qualifier("pricing-db") DataSource pricingDataSource;
```

## \<cloud:properties\>

The `<cloud:properties>` element exposes basic information about the application and its bound services as properties.  Your application can then consume these properties using the Spring property placeholder support.

The `<cloud:properties>` element has just a single attribute (`id`) which specifies the name of the Properties bean.  Use this ID as a reference to `<context:property-placeholder>` which you can use to hold all the properties exposed by Cloud Foundry.  You can then use these properties in your other bean defintions.

Note that if you are using Spring Framework 3.1 (or later), these properties are automatically available without having to include `<cloud:properties>` in your application context file.

The following example shows how to use this element in your Spring application context file:

``` xml
    <cloud:properties id="cloudProperties" />

    <context:property-placeholder properties-ref="cloudProperties" />

    <bean class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="user"
                  value="${cloud.services.mysql.connection.username}" />
    ...
    </bean>
```

In the preceding example, `cloud.services.mysql.connection.username` is one of the properties exposed by Cloud Foundry.

For a complete list of properties exposed by Cloud Foundry, as well as a more detailed example, see [Accessing Cloud Foundry Properties](#accessing-cloud-foundry-properties).

## RabbitMQ And Spring: Additional Programming Information

This section provides additional information about using RabbitMQ in your Spring applications that you deploy to Cloud Foundry. This section is not intended to be a complete tutorial on RabbitMQ and Spring; for that, see the following resources:

+	[RabbitMQ Tutorials](http://www.rabbitmq.com/getstarted.html), cover the basics of creating messaging in your applications.
+	[Download](http://www.rabbitmq.com/download.html), [install](http://www.rabbitmq.com/install.html) and [configure](http://www.rabbitmq.com/configure.html) RabbitMQ
+       [Spring AMPQ reference documentation](http://static.springsource.org/spring-amqp/docs/1.0.x/reference/html/)

The RabbitMQ service is accessed through the [AMQP protocol](http://www.amqp.org/) (versions 0.8 and 0.9.1) and your application will need access to a AMQP client library in order to use the service. The Spring AMQP project enables AMQP applications to be built using Spring constructs.

The following sample `pom.xml` file shows RabbitMQ dependencies and repositories in addition to the `cloudfoundry-runtime` dependency described above:

```xml
    <repositories>
        <repository>
              <id>org.springframework.maven.milestone</id>
               <name>Spring Maven Milestone Repository</name>
               <url>http://maven.springframework.org/milestone</url>
               <snapshots>
                       <enabled>false</enabled>
               </snapshots>
        </repository>
    </repositories>

    <dependency>
        <groupId>cglib</groupId>
        <artifactId>cglib-nodep</artifactId>
        <version>2.2</version>
    </dependency>

    <dependency>
        <groupId>org.springframework.amqp</groupId>
        <artifactId>spring-rabbit</artifactId>
        <version>1.0.0.RC2</version>
    </dependency>

    <dependency>
        <groupId>org.cloudfoundry</groupId>
        <artifactId>cloudfoundry-runtime</artifactId>
        <version>0.7.1</version>
    </dependency>
```

Then update your application controller/logic as follows:

* Include the messaging libraries:

```java
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.amqp.core.AmqpTemplate;
```

* Read and write messages as shown in the following Java code snippets:

``` java
   @Controller
   public class HomeController {
       @Autowired AmqpTemplate amqpTemplate;

       @RequestMapping(value = "/")
       public String home(Model model) {
           model.addAttribute(new Message());
           return "WEB-INF/views/home.jsp";
       }

       @RequestMapping(value = "/publish", method=RequestMethod.POST)
       public String publish(Model model, Message message) {
           // Send a message to the "messages" queue
           amqpTemplate.convertAndSend("messages", message.getValue());
           model.addAttribute("published", true);
           return home(model);
       }

       @RequestMapping(value = "/get", method=RequestMethod.POST)
       public String get(Model model) {
           // Receive a message from the "messages" queue
           String message = (String)amqpTemplate.receiveAndConvert("messages");
           if (message != null)
               model.addAttribute("got", message);
           else
               model.addAttribute("got_queue_empty", true);

           return home(model);
   }
```

## Using Spring Profiles to Conditionalize Cloud Foundry Configuration

The preceding sections describe how to use the `<cloud:>` namespace to easily configure services (such as data sources and RabbitMQ connection factories) for Spring applications deployed to Cloud Foundry.   However, you might not always want to deploy your applications to Cloud Foundry; for example, you might sometimes want to use a local environment to test the application during iterative development.  In this case, it would be useful to *conditionalize* the application configuration so that only specific fragments are activated when a certain condition is true.   Setting up such conditionalized configuration makes your application portable to many different environments so that you do not have to manually change the configuration when you deploy it to, for example, your local environment and then to Cloud Foundry.   To enable this, use the Spring *profiles* feature, available in Spring Framework 3.1 or later.

The basic idea is that you group the configuration for a specific environment using the `profile` attribute of a nested `<beans>` element in the appropriate Spring application context file.  You can create your own custom profiles, but the ones that are most relevant in the context of Cloud Foundry are `default` and `cloud`.

When you deploy a Spring application to Cloud Foundry, Cloud Foundry automatically enables the `cloud` profile. This allows for a pre-defined, convenient location for Cloud Foundry-specific application configuration. You should then group all specific usages of the `<cloud:>` namespace within the `cloud` profile block to allow the application to run outside of Cloud Foundry environments.  You then use the `default` profile (or a custom profile) to group the non-Cloud Foundry configuration that will be used if you deploy your application to a non-Cloud Foundry environment.

Here is an example of a Spring `MongoTemplate` being populated from two alternately configured connection factories. When running on Cloud Foundry (`cloud` profile), the connection factory is automatically configured. When not running on Cloud Foundry (`default` profile), the connection factory is manually configured with the connection settings to a running MongoDB instance.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans  xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:cloud="http://schema.cloudfoundry.org/spring"
        xmlns:jdbc="http://www.springframework.org/schema/jdbc"
        xmlns:util="http://www.springframework.org/schema/util"
        xmlns:mongo="http://www.springframework.org/schema/data/mongo"
        xsi:schemaLocation="http://www.springframework.org/schema/data/mongo
          http://www.springframework.org/schema/data/mongo/spring-mongo-1.0.xsd
          http://www.springframework.org/schema/jdbc
          http://www.springframework.org/schema/jdbc/spring-jdbc-3.1.xsd
          http://schema.cloudfoundry.org/spring
          http://schema.cloudfoundry.org/spring/cloudfoundry-spring.xsd
          http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
          http://www.springframework.org/schema/util
          http://www.springframework.org/schema/util/spring-util-3.1.xsd">

        <bean id="mongoTemplate" class="org.springframework.data.mongodb.core.MongoTemplate">
           <constructor-arg ref="mongoDbFactory" />
        </bean>

        <beans profile="default">
           <mongo:db-factory id="mongoDbFactory" dbname="pwdtest" host="127.0.0.1" port="27017" username="test_user" password="efgh" />
        </beans>

        <beans profile="cloud">
           <cloud:mongo-db-factory id="mongoDbFactory" />
        </beans>

</beans>
```

Note that the `<beans profile="value">` element is nested inside the standard root `<beans>` element.  The MongoDB connection factory in the `cloud` profile uses the `<cloud:>` namespace, the connection factory configuration in the `default` profile uses the `<mongo:>` namespace.   You can now deploy this application to the two different environments without making any manual changes to its configuration when you switch from one to the other.

See the [SpringSource Blog](http://blog.springsource.com/2011/02/11/spring-framework-3-1-m1-released/) for additional information about using Spring Profiles, a new feature in Spring Framework 3.1.

## Sending Email From Spring Applications Deployed to Cloud Foundry

In order to prevent spam and other abuse, SMTP is blocked from applications running in Cloud Foundry.  However, your applications can still send email when deployed to Cloud Foundry, as described in this section.

Service providers, such as [SendGrid](http://sendgrid.com), can send email on your behalf via HTTP web services, which is an option you can use when deploying your application to Cloud Foundry.  If, however, you also run your application inside your datacenter, you might want to use your corporate SMTP servers in that case.  This is a good example of using [Spring Profiles](#using-spring-profiles-to-conditionalize-cloud-foundry-configuration) to conditionalize how your application sends email, depending on where exactly the application happens to be deployed.  This makes your application portable to different environments without you having to manually update its configuration.

The following snippet of a Spring application context shows how to specify that your application should use SendGrid to send email when the application is running in Cloud Foundry; note the use of the `cloud` profile:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans  xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:cloud="http://schema.cloudfoundry.org/spring"

    ...

    <beans profile="cloud">
       <bean name="mailSender" class="example.SendGridMailSender">
          <property name="apiUser" value="youremail@domain.com" />
          <property name="apiKey" value="secureSecret" />
       </bean>
    </beans>

   ...

    <!-- additional beans in your application -->

</beans>
```

In the example, `example.SendGridMailSender` is the Spring bean that sends email using the SendGrid service provider; however, this bean will only be activated when your application is deployed to Cloud Foundry.  If your application is actually running inside your datacenter, then your default email server is used.

## Accessing Cloud Foundry Properties

Cloud Foundry exposes a number of application and service properties directly into its deployed applications.  Your deployed application can in turn consume these properties.   The properties exposed by Cloud Foundry include basic information about the application, such as its name and the Cloud provider, and detailed connection information for all services currently bound to the application.

Service properties generally take one of the following forms:

        cloud.services.{service-name}.connection.{property}
        cloud.services.{service-name}.{property}

where `{service-name}` refers to the name you gave the service when you bound it to your application at deploy time.  The specific connection properties that are available depend on the type of service; see the table at the end of this section for the complete list.

For example, assume that in VMC you created a vFabric Postgres service called `my-postgres` and then bound it to your application; Cloud Foundry exposes the following properties about this service that your application in turn can consume:

        cloud.services.my-postgres.connection.host
        cloud.services.my-postgres.connection.hostname
        cloud.services.my-postgres.connection.name
        cloud.services.my-postgres.connection.password
        cloud.services.my-postgres.connection.port
        cloud.services.my-postgres.connection.user
        cloud.services.my-postgres.connection.username
        cloud.services.my-postgres.plan
        cloud.services.my-postgres.type

For convenience, if you have bound just *one* service of a given type to your application, Cloud Foundry creates an alias based on the service type instead of the service name. For example, if only one MySQL service is bound to an application, the properties will take the form `cloud.services.mysql.connection.{property}`.   Cloud Foundry uses the following aliases in this case:

+ `mysql`
+ `mongodb`
+ `postgresql`
+ `rabbitmq`
+ `redis`

If you want to use these Cloud Foundry properties in your application, use a Spring property placholder inside of a `cloud` profile; Spring profiles are briefly described in [Using Spring Profiles to Conditionalize Cloud Foundry Configuration](#using-spring-profiles-to-conditionalize-cloud-foundry-configuration).

For example, assume that you have bound a MySQL service called `spring-mysql` to your application, but your application requires a c3p0 connection pool instead of the connection pool provided by Cloud Foundry.   But you still want to use the same connection properties defined by Cloud Foundry for the MySQL service, in particular the username and password and JDBC URL.  The following Spring application context snippet shows how you might implement this:

```xml
<beans profile="cloud">
   <bean id="c3p0DataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource" destroy-method="close">
      <property name="driverClass" value="com.mysql.jdbc.Driver" />
      <property name="jdbcUrl"
                value="jdbc:mysql://${cloud.services.spring-mysql.connection.host}:${cloud.services.spring-mysql.connection.port}/${cloud.services.spring-mysql.connection.name}" />
      <property name="user" value="${cloud.services.spring-mysql.connection.username}" />
      <property name="password" value="${cloud.services.spring-mysql.connection.password}" />
   </bean>
</beans>
```

The following table lists all the application and service properties that Cloud Foundry exposes to deployed applications.  In the property names, `{service-name}` refers to the actual name of the bound service.

<table class="std">
 <tr>
   <th>Property</th>
   <th>Associated Service Type</th>
   <th>Description</th>
 </tr>
 <tr>
   <td>cloud.application.name</td>
   <td>Not applicable.</td>
   <td>The name of the application.</td>
 </tr>
 <tr>
   <td>cloud.provider.url</td>
   <td>Not applicable.</td>
   <td>The URL of the cloud hosting your application, such as <tt>cloudfoundry.com</tt>.</td>
 </tr>
 <tr>
   <td>cloud.services.{<i>service-name</i>}.connection.db</td>
   <td>MongoDB</td>
   <td>Name of the database that Cloud Foundry created.</td>
 </tr>
 <tr>
   <td>cloud.services.{<i>service-name</i>}.connection.host</td>
   <td>MongoDB</td>
   <td>Name or IP address of the host on which the MongoDB server is running.</td>
 </tr>
 <tr>
   <td>cloud.services.{<i>service-name</i>}.connection.hostname</td>
   <td>MongoDB</td>
   <td>Name or IP address of the host on which the MongoDB server is running.</td>
 </tr>
 <tr>
   <td>cloud.services.{<i>service-name</i>}.connection.name</td>
   <td>MongoDB</td>
   <td>Name of the user that connects to the MongoDB database.</td>
 </tr>
 <tr>
   <td>cloud.services.{<i>service-name</i>}.connection.password</td>
   <td>MongoDB</td>
   <td>Password of the user that connects to the MongoDB database.</td>
 </tr>
 <tr>
   <td>cloud.services.{<i>service-name</i>}.connection.port</td>
   <td>MongoDB</td>
   <td>Listen port of the MongoDB server.</td>
 </tr>
 <tr>
   <td>cloud.services.{<i>service-name</i>}.connection.username</td>
   <td>MongoDB</td>
   <td>Name of the user that connects to the MongoDB database.</td>
 </tr>
 <tr>
   <td>cloud.services.{<i>service-name</i>}.plan</td>
   <td>MongoDB</td>
   <td>Pay plan for the service, such as <tt>free</tt>.</td>
 </tr>
 <tr>
   <td>cloud.services.{<i>service-name</i>}.type</td>
   <td>MongoDB</td>
   <td>Name and version of the MongoDB server.</td>
 </tr>
 <tr>
   <td>cloud.services.{<i>service-name</i>}.connection.name</td>
   <td>MySQL</td>
   <td>Name of the MySQL database that Cloud Foundry created.</td>
 </tr>
 <tr>
   <td>cloud.services.{<i>service-name</i>}.connection.host</td>
   <td>MySQL</td>
   <td>Name or IP address of the host on which the MySQL server is running.</td>
 </tr>
 <tr>
   <td>cloud.services.{<i>service-name</i>}.connection.hostname</td>
   <td>MySQL</td>
   <td>Name or IP address of the host on which the MySQL server is running.</td>
 </tr>
 <tr>
   <td>cloud.services.{<i>service-name</i>}.connection.port</td>
   <td>MySQL</td>
   <td>Listen port of the MySQL server.</td>
 </tr>
 <tr>
   <td>cloud.services.{<i>service-name</i>}.connection.user</td>
   <td>MySQL</td>
   <td>Name of the user that connects to the MySQL database.</td>
 </tr>
 <tr>
   <td>cloud.services.{<i>service-name</i>}.connection.username</td>
   <td>MySQL</td>
   <td>Name of the user that connects to the MySQL database.</td>
 </tr>
 <tr>
   <td>cloud.services.{<i>service-name</i>}.connection.password</td>
   <td>MySQL</td>
   <td>Password of the user that connects to the MySQL database.</td>
 </tr>
 <tr>
   <td>cloud.services.{<i>service-name</i>}.plan</td>
   <td>MySQL</td>
   <td>Pay plan for the service, such as <tt>free</td>.</td>
 </tr>
 <tr>
   <td>cloud.services.{<i>service-name</i>}.type</td>
   <td>MySQL</td>
   <td>Name and version of the MySQL server.</td>
 </tr>
 <tr>
   <td>cloud.services.{<i>service-name</i>}.connection.name</td>
   <td>vFabric Postgres</td>
   <td>Name of the vFabric Postgres database that Cloud Foundry created.</td>
 </tr>
 <tr>
   <td>cloud.services.{<i>service-name</i>}.connection.host</td>
   <td>vFabric Postgres</td>
   <td>Name or IP address of the host on which the vFabric Postgres server is running.</td>
 </tr>
 <tr>
   <td>cloud.services.{<i>service-name</i>}.connection.hostname</td>
   <td>vFabric Postgres</td>
   <td>Name or IP address of the host on which the vFabric Postgres server is running.</td>
 </tr>
 <tr>
   <td>cloud.services.{<i>service-name</i>}.connection.port</td>
   <td>vFabric Postgres</td>
   <td>Listen port of the vFabric Postgres server.</td>
 </tr>
 <tr>
   <td>cloud.services.{<i>service-name</i>}.connection.user</td>
   <td>vFabric Postgres</td>
   <td>Name of the user that connects to the vFabric Postgres database.</td>
 </tr>
 <tr>
   <td>cloud.services.{<i>service-name</i>}.connection.username</td>
   <td>vFabric Postgres</td>
   <td>Name of the user that connects to the vFabric Postgres database.</td>
 </tr>
 <tr>
   <td>cloud.services.{<i>service-name</i>}.connection.password</td>
   <td>vFabric Postgres</td>
   <td>Password of the user that connects to the vFabric Postgres database.</td>
 </tr>
 <tr>
   <td>cloud.services.{<i>service-name</i>}.plan</td>
   <td>vFabric Postgres</td>
   <td>Pay plan for the service, such as <tt>free</tt>.</td>
 </tr>
 <tr>
   <td>cloud.services.{<i>service-name</i>}.type</td>
   <td>vFabric Postgres</td>
   <td>Name and version of the vFabric Postgres server.</td>
 </tr>
 <tr>
   <td>cloud.services.{<i>service-name</i>}.connection.url</td>
   <td>RabbitMQ</td>
   <td>URL used to connect to the AMPQ broker.  URL includes the host, port, username, and so on.</td>
 </tr>
 <tr>
   <td>cloud.services.{<i>service-name</i>}.plan</td>
   <td>RabbitMQ</td>
   <td>Pay plan for the service, such as <tt>free</td>.</td>
 </tr>
 <tr>
   <td>cloud.services.{<i>service-name</i>}.type</td>
   <td>RabbitMQ</td>
   <td>Name and version of the RabbitMQ server.</td>
 </tr>
 <tr>
   <td>cloud.services.{<i>service-name</i>}.connection.host</td>
   <td>Redis</td>
   <td>Name or IP address of the host on which the Redis server is running.</td>
 </tr>
 <tr>
   <td>cloud.services.{<i>service-name</i>}.connection.hostname</td>
   <td>Redis</td>
   <td>Name or IP address of the host on which the Redis server is running.</td>
 </tr>
 <tr>
   <td>cloud.services.{<i>service-name</i>}.connection.port</td>
   <td>Redis</td>
   <td>Listen port of the Redis server.</td>
 </tr>
 <tr>
   <td>cloud.services.{<i>service-name</i>}.connection.name</td>
   <td>Redis</td>
   <td>Name of the user that connects to the Redis database.</td>
 </tr>
 <tr>
   <td>cloud.services.{<i>service-name</i>}.connection.password</td>
   <td>Redis</td>
   <td>Password of the user that connects to the Redis database.</td>
 </tr>
 <tr>
   <td>cloud.services.{<i>service-name</i>}.plan</td>
   <td>Redis</td>
   <td>Pay plan for the service, such as <tt>free</tt>.</td>
 </tr>
 <tr>
   <td>cloud.services.{<i>service-name</i>}.type</td>
   <td>Redis</td>
   <td>Name and version of the Redis server.</td>
 </tr>
</table>
