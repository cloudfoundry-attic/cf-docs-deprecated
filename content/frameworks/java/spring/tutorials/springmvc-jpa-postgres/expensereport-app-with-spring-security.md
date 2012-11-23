---
title: Spring MVC App with Spring Security
description: Adding Spring Security to the ExpenseReport App
tags:
    - spring
    - sts
    - tomcat
    - maven
    - spring-security
---

## Introduction
Spring Security is a customizable authentication and access service framework for server side Java-based enterprise software applications. The Spring security OAuth provides a method for making authenticated HTTP requests using a token - an identifier used to denote an access grant with specific scope, duration, and other attributes. Tokens are issued to third-party clients by an authorization server with the approval of the resource owner.  Instead of sharing their credentials with the client, resource owners grant access by authenticating directly with the authorization server which in turn issues a token to the client. The client uses the token (and optional secret) to authenticate with the resource server and gain access.

  For example, a web user(resource owner) can grant a printing service (client) access to his protected photos stored at a photo sharing service (resource server), without sharing his username and password with the printing service. Instead, he authenticates directly with the photo sharing service (authorization server) which issues the printing service delegation-specific credentials.

## Prerequisites
Before you begin this tutorial, you should:

1.  Be familiar working with STS or Eclipse.

### Setup: Exercise3-Starter
Import the source code from the downloaded folder Exercise3-Starter as given. Open STS, Select **File > Import > Maven > Existing Maven Projects** and select Exercise3-Starter folder.

  ![maven-import-step1](/images/spring_tutorial/import-maven-project-step1.png)

  ![maven-import-step2](/images/spring_tutorial/import-maven-project-step3.png)

## Adding Spring Security to the ExpenseReport App
This section will teach you how to integrate Spring Security in to the ExpenseReport Application. These are the required dependencies to be added in pom.xml:
```xml
<dependency>
 <groupId>org.springframework.security</groupId>
 <artifactId>spring-security-core</artifactId>
 <version>3.0.5.RELEASE</version>
</dependency>
 
<dependency>
 <groupId>org.springframework.security</groupId>
 <artifactId>spring-security-web</artifactId>
 <version>3.0.5.RELEASE</version>
</dependency>
  
<dependency>
 <groupId>org.springframework.security</groupId>
 <artifactId>spring-security-config</artifactId>
 <version>3.0.5.RELEASE</version>
</dependency>
```

## Enabling Spring Security
Go over the following steps to enable Spring Security:

1.     Add a **DelegatingFilterProxy** in `src/main/webapp/WEB-INF/web.xml`

2.     Create a custom XML configuration named spring-security.xml in `src/main/webapp/WEB-INF`

In web.xml, declare an instance of a DelegatingFilterProxy. This will filter the requests depending on the declared url-pattern.

```xml
<filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>
<filter-mapping>
    <filter-name>springSecurityFilterChain</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

In spring-security.xml, define Web/HTTP security configuration by adding an http tag. Also define the url pattern to be intecepted. Add the "jdbc-user service" tag and define the query to get the user and authorities data from the database.

1.  To access the application, the user must be either `ROLE_USER` or `ROLE_MANAGER`. To validate this function, add `<intercept-url pattern="/" access="hasAnyRole('ROLE_USER','ROLE_MANAGER')"/>`.

2.  Since only the manager is allowed to approve expenses, restrict access to him alone by adding `<intercept-url pattern="/expenses/approvals" access="hasRole('ROLE_MANAGER')"/>`.

```xml
<beans:beans xmlns="http://www.springframework.org/schema/security"
    xmlns:beans="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
    http://www.springframework.org/schema/security
    http://www.springframework.org/schema/security/spring-security-3.0.3.xsd">
    
    <http auto-config="true" use-expressions="true">
        <intercept-url pattern="/login" filters="none" />
        <intercept-url pattern="/logout" filters="none" />
        <intercept-url pattern="/signup**" filters="none" />
        <intercept-url pattern="/resources/**" filters="none" />
        <intercept-url pattern="/favicon**" filters="none" />
        <intercept-url pattern="/" access="hasAnyRole('ROLE_USER','ROLE_MANAGER')"/>
        <intercept-url pattern="/expenses/approvals" access="hasRole('ROLE_MANAGER')" />
        <intercept-url pattern="/expenses/**/state**" access="hasRole('ROLE_MANAGER')" />
        <intercept-url pattern="/**" access="isAuthenticated()" />
        <form-login login-page="/login" default-target-url="/"
            authentication-failure-url="/loginfailed" />
        <logout logout-success-url="/logout" />
        <session-management invalid-session-url="/sessiontimeout" >
            <concurrency-control max-sessions="1" />
        </session-management>
    </http>
    <authentication-manager>
        <authentication-provider>
            <jdbc-user-service data-source-ref="dataSource"
               users-by-username-query="
                     select username,password,enabled
                     from users where username=?"
               authorities-by-username-query="
                     select u.username, r.rolename from users u, role r
                     where u.role_roleid = r.roleid and u.username =?  " />
        </authentication-provider>
    </authentication-manager>
</beans:beans>
```

To load this file as a part of configuration add **@ImportResource("/WEB-INF/spring-security.xml")** in ComponentConfig class.

Create LoginController in the `com.springsource.html5expense.contoller` package and add mapping for login,logout,loginfailed,signup. The login.jsp has two input elements for username and password(j_username and j_password). These are Spring's placeholder names for username and password respectively.
When the form is submitted, it will be sent to the following action URL: j_spring_security_check.

Find the LoginController code [here](/frameworks/java/spring/tutorials/springmvc-jpa-postgres/code/logincontroller.html).

## Authenication Flow
* Once you have added Spring security into the Expense Reporting App, it will ensure that the user has been authenticated and authorized to access the resource. The below figure shows how the user interacts with the application.

    ![sequence-flow](/images/spring_tutorial/Spring-security-flow.png)

## Check Point
1. Select project, drag it down and drop into your VMware vFabric tc server listed in the bottom left Server window.

2. Select VMware vFabric tc server and press **CTRL+ALT+R** to run the server.

3. Once the Server starts, open your browser and enter application url : `http://localhost:8080/html5expense/expenses`. This time you will be automatically redirected to the login page.

    ![STS Fabric Server.png](/images/spring_tutorial/localhost_login.png)

## Complete Application Code
If you are getting any errors, download the Exercise3-Complete code and import into STS.

## Deploying to Cloud Foundry
* To learn more about deploying Spring App with PostgreSQL to Cloud Foundry, please refer [here](/frameworks/java/spring/tutorials/springmvc-jpa-postgres/springmvc-app-with-postgresql-deployment-to-cloudfoundry.html).
 
<a class="button-plain" style="padding: 3px 15px;" href="/frameworks/java/spring/tutorials/springmvc-jpa-postgres/spring-expensereport-app-tutorial.html">Prev</a>
