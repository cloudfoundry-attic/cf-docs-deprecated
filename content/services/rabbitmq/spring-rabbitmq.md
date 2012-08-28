---
title: Using RabbitMQ with Spring
description: Getting Started with the RabbitMQ Service from a Spring Application
tags:
    - spring
    - rabbitmq
    - tutorial
    - code-attached
---

This tutorial explains how to use the RabbitMQ service for Cloud Foundry from a Java and Spring application.

There are a number of ways that messaging can be useful in the context of a Spring web application. See the [Getting Started](http://www.rabbitmq.com/getstarted.html) area on the main RabbitMQ web site for some ideas. But in this tutorial, we'll build a very simple application that uses RabbitMQ, focusing on the basics of connecting to the RabbitMQ service. Once you understand that, you will be able to integrate more realistic uses of the service into your Spring applications on Cloud Foundry.

This tutorial goes through the complete process of creating a very simple Spring application on Cloud Foundry, and hooking it up to the RabbitMQ service. If you already have your own applications on Cloud Foundry, much of this material may already be familiar. If so, you can skip straight to the last section, where we discuss how to access the RabbitMQ service from your application. You can also find the complete code for the finished application [on GitHub](https://github.com/rabbitmq/rabbitmq-coudfoundry-samples/tree/master/spring).

The application we are going to build will consist of a single page, looking like this:

![page.png](http://support.cloudfoundry.com/attachments/token/kx0fcgzrgasd4eq/?name=page.png)

This application can publish messages to and get messages from a single RabbitMQ queue. We can type a message into the input box and click the Publish button to publish it to the queue. Alternatively, we can click the Get button to take a message from the queue; the next message will be displayed, or it will say that the queue was empty.


## Prerequisites

To start off, there are a few things that need to be installed on your development machine. This guide will assume that you already have the [JDK 6](http://www.oracle.com/technetwork/java/javase/downloads/) and [Maven](http://maven.apache.org/) installed.

This tutorial will use the vmc command line tool used to interact with Cloud Foundry. If you don't already have vmc installed, please follow the instructions [here](/tools/vmc/vmc.html). Note that vmc continues to be enhanced, so even if you already have it installed, you should update it regularly by doing:

```bash
$ gem update vmc
```

vmc is not the only way to develop with Cloud Foundry: [Spring Tool Suite](http://www.springsource.org/sts) supports Clound Foundry in an Eclipse-powered development environment. See [this section](http://www.springsource.org/sts) for how to use Cloud Foundry within STS.

Creating an Initial Spring MVC App
Now we will create a minimal Spring VMC app and push it to Cloud Foundry, following the article [Green Beans: Getting Started with Spring MVC](http://blog.springsource.org/2011/01/04/green-beans-getting-started-with-spring-mvc/). The application source comprises just five files. The file locations will obey the normal Maven project layout.

src/main/webapp/WEB-INF/web.xml simply hands off to the Spring framework:

```xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">
    <servlet>
        <servlet-name>appServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/spring/servlet-context.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>appServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

The Spring Application Context XML file, src/main/webapp/WEB-INF/spring/servlet-context.xml, is minimal, simply declaring that we will use annotation-based configuration for MVC, and which package to scan:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-3.0.xsd
                           http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.0.xsd">
    <context:component-scan base-package="com.rabbitmq.cftutorial.simple"/>
    <mvc:annotation-driven/>
</beans>
```


The controller class, src/main/java/com/rabbitmq/cftutorial/simple/HomeController.java, simply hands off to the JSP template:

```java
package com.rabbitmq.cftutorial.simple;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class HomeController {
    @RequestMapping(value = "/")
    public String home() {
        return "WEB-INF/views/home.jsp";
    }
}
```

And the JSP template, src/main/webapp/WEB-INF/views/home.jsp, displays a simple static page.

```jsp
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ page session="false" %>
<html>
  <head>
    <title>Simple RabbitMQ Application</title>
  </head>
  <body>
    <h1>Simple RabbitMQ Application</h1>
  </body>
</html>
```

Finally, the pom.xml file to describe the project to Maven:

```xml
 <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.rabbitmq.cftutorial</groupId>
  <artifactId>simple</artifactId>
  <packaging>war</packaging>
  <version>1.0-SNAPSHOT</version>
  <name>cftutorial-simple</name>
  <description>Simple RabbitMQ Application</description>

  <properties>
      <java-version>1.6</java-version>
      <org.springframework-version>3.0.5.RELEASE</org.springframework-version>
  </properties>

  <dependencies>
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-context</artifactId>
          <version>${org.springframework-version}</version>
      </dependency>
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-webmvc</artifactId>
          <version>${org.springframework-version}</version>
      </dependency>

      <dependency>
          <groupId>javax.servlet</groupId>
          <artifactId>jstl</artifactId>
          <version>1.2</version>
      </dependency>
  </dependencies>

  <build>
      <plugins>
          <plugin>
              <groupId>org.apache.maven.plugins</groupId>
              <artifactId>maven-compiler-plugin</artifactId>
              <configuration>
                  <source>${java-version}</source>
                  <target>${java-version}</target>
              </configuration>
          </plugin>
      </plugins>
  </build>
</project>
```

Next we build the project:

```bash
$ mvn package
[...]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESSFUL
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 2 seconds
[INFO] Finished at: Tue Aug 02 14:04:59 BST 2011
[INFO] Final Memory: 17M/321M
[INFO] ------------------------------------------------------------------------
```

And we are ready to push the application to Cloud Foundry. Change to the target directory before doing vmc push. Here, I will call my application “rabbit-simple”, but you should choose your own name.

We also get the opportunity to bind services to our application at this point. But we won't add the rabbitmq service at this point — we will do that later, after we have the initial app running.

```bash
$ cd target
$ vmc push
Would you like to deploy from the current directory? [Yn]: y
Application Name: rabbit-simple
Application Deployed URL: 'rabbit-simple.cloudfoundry.com'?
Detected a Java SpringSource Spring Application, is this correct? [Yn]: y
Memory Reservation [Default:512M] (64M, 128M, 256M or 512M)
Creating Application: OK
Would you like to bind any services to 'rabbit-simple'? [yN]: n
Uploading Application:
  Checking for available resources: OK
  Processing resources: OK
  Packing application: OK
  Uploading (3K): OK
Push Status: OK
Staging Application: OK
Starting Application: OK
```

If this succeeds, you should be able to go to the URL for the application and see the home page:

![home.png](http://support.cloudfoundry.com/attachments/token/119cuj9gvkhnzs4/?name=home.png)

## Extending the application

Now that we have a minimal Spring MVC application running on Cloud Foundry, we will add the pieces we need to demonstrate the use of the RabbitMQ service.

We extend the JSP template at src/main/webapp/WEB-INF/views/home.jsp with the page elements to construct the page shown in the introduction:

```jsp
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<%@ page session="false" %>
<html>
  <head>
    <title>Simple RabbitMQ Application</title>
  </head>
  <body>
    <h1>Simple RabbitMQ Application</h1>

    <h2>Publish a message</h2>

    <form:form modelAttribute="message" action="/publish" method="post">
      <form:label for="value" path="value">Message to publish:</form:label>
      <form:input path="value" type="text"/>

      <input type="submit" value="Publish"/>
    </form:form>

    <c:if test="${published}">
      <p>Published a message!</p>
    </c:if>

    <h2>Get a message</h2>

    <form:form action="/get" method="post">
      <input type="submit" value="Get one"/>
    </form:form>

    <c:choose>
      <c:when test="${got_queue_empty}">
        <p>Queue empty</p>
      </c:when>
      <c:when test="${got != null}">
        <p>Got message: <c:out value="${got}"/></p>
      </c:when>
    </c:choose>
  </body>
</html>
```

And we need to add actions to the controller class to support the Publish and Get one forms:

```java
package com.rabbitmq.cftutorial.simple;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.ui.Model;

@Controller
public class HomeController {
    @RequestMapping(value = "/")
    public String home(Model model) {
        model.addAttribute(new Message());
        return "WEB-INF/views/home.jsp";
    }

    @RequestMapping(value = "/publish", method=RequestMethod.POST)
    public String publish(Model model) {
        return home(model);
    }

    @RequestMapping(value = "/get", method=RequestMethod.POST)
    public String get(Model model) {
        return home(model);
    }
}
```

We also need to add a Message class, to back the Publish form. This simply holds the String value for the message.

```java
package com.rabbitmq.cftutorial.simple;

public class Message {
    private String value;

    public String getValue() {
        return value;
    }

    public void setValue(String value) {
        this.value = value;
    }
}
```

With these additions, we can update the application on Cloud Foundry and take a look:

```bash
$ mvn package
[...]
$ cd target
$ vmc update rabbit-simple
Uploading Application:
  Checking for available resources: OK
  Processing resources: OK
  Packing application: OK
  Uploading (3K): OK
Push Status: OK
Stopping Application: OK
Staging Application: OK
Starting Application: OK
```

You should now be able to see the page shown in the introduction above. In the next section we will fill out the code for the actions in the controller in order to make the application fully functional.

## Using the RabbitMQ Service

In this section, we will take the simple application created in the previous section and make it use the RabbitMQ service.

First, we will create an instance of the RabbitMQ service on Cloud Foundry:

```bash
$ vmc create-service
1. rabbitmq
2. mysql
3. mongodb
4. redis
Please select one you wish to provision: 1
Creating Service [rabbitmq-aaaad]: OK
```

At this point, the service instance has been successfully created. But in order for it to be accessible to our application, we also need to bind it:

```bash
$ vmc bind-service rabbitmq-aaaad rabbit-simple
Binding Service: OK
Stopping Application: OK
Staging Application: OK
Starting Application: OK
```

The vmc apps command confirms that the service has been bound to our application:

```bash
$ vmc apps

+-----------------+----+---------+----------------------------------+-----------------------------+
| Application     | #  | Health  | URLS                             | Services                    |
+-----------------+----+---------+----------------------------------+-----------------------------+
| rabbit-simple   | 1  | RUNNING | rabbit-simple.cloudfoundry.com   | rabbitmq-aaaad              |
+-----------------+----+---------+----------------------------------+-----------------------------+
```

The RabbitMQ service is accessed through the [AMQP](http://www.amqp.org/) protocol (RabbitMQ supports versions 0.8 and 0.9.1 of AMQP). So we need to use an AMQP client library. Fortunately, there is a Spring AMQP project which allows AMQP applications to take advantage of Spring concepts. We will also use the cloudfoundry-runtime jar, which facilitates access to Cloud Foundry services, including RabbitMQ, from Spring applications. So we will add the corresponding dependencies to the pom.xml file:

```xml
[...]
   <properties>
       [...]
   </properties>

  <repositories>
      <repository>
          <id>spring-milestone</id>
          <name>Spring Maven MILESTONE Repository</name>
          <url>http://maven.springframework.org/milestone</url>
      </repository>
  </repositories>

  <dependencies>
      [...]
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
      [...]
  </dependencies>
[...]
```

Next we extend the Spring Application Context XML. The additions:

+ Use cloudfoundry-runtimeto obtain a connection to the RabbitMQ service.
+ Configure the RabbitTemplate and RabbitAdmin that are the main entry points to Spring AMQP.
+ Declare a queue called messages within the RabbitMQ broker.
+ See the [Spring AMQP documentation](http://static.springsource.org/spring-amqp/docs/1.0.x/reference/html/) for more details.

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:rabbit="http://www.springframework.org/schema/rabbit"
       xmlns:cloud="http://schema.cloudfoundry.org/spring"
       xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-3.0.xsd
                           http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.0.xsd
                           http://www.springframework.org/schema/rabbit http://www.springframework.org/schema/rabbit/spring-rabbit-1.0.xsd
                           http://schema.cloudfoundry.org/spring http://schema.cloudfoundry.org/spring/cloudfoundry-spring-0.7.xsd">
    <context:component-scan base-package="com.rabbitmq.cftutorial.simple"/>
    <mvc:annotation-driven/>

    <!-- Obtain a connection to the RabbitMQ via cloudfoundry-runtime: -->
    <cloud:rabbit-connection-factory id="connectionFactory"/>

    <!-- Set up the AmqpTemplate/RabbitTemplate: -->
    <rabbit:template connection-factory="connectionFactory"/>

    <!-- Request that queues, exchanges and bindings be automatically
         declared on the broker: -->
    <rabbit:admin connection-factory="connectionFactory"/>

    <!-- Declare the "messages" queue: -->
    <rabbit:queue name="messages" durable="true"/>
</beans>
```

Finally, the changes to HomeController to actually publish and get messages are very simple:

```java
package com.rabbitmq.cftutorial.simple;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.ui.Model;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.amqp.core.AmqpTemplate;

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
}
```

Everything is now in place, so a final vmc update gives us the fully functional application. We can publish a message:

![publish1.png](http://support.cloudfoundry.com/attachments/token/6btegldivmhssve/?name=publish1.png)
![publish2.png](http://support.cloudfoundry.com/attachments/token/y4ls3aqh00thzen/?name=publish2.png)

And get it back:

![get1.png](http://support.cloudfoundry.com/attachments/token/e5ojjwwhs6ljjwd/?name=get1.png)
![get2.png](http://support.cloudfoundry.com/attachments/token/lejqlsjmiu75lc7/?name=get2.png)

That concludes this tutorial. You can find more resources about RabbitMQ and AMQP on the [RabbitMQ web site](http://www.rabbitmq.com/). And you can find more information about using RabbitMQ within a Spring application on the [Spring AMQP](http://www.springsource.org/spring-amqp) website. And if you have questions or feedback on this tutorial or the RabbitMQ service, please contact us using the forums on [support.cloudfoundry.com](http://support.cloudfoundry.com/).