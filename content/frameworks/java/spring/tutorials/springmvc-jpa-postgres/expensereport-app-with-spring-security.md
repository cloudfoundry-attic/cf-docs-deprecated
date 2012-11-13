---
title: Spring MVC App with Spring Security
description: Adding Spring Security to the ExpenseReport App
tags:
    - Spring
    - STS
    - Tomcat
    - Maven
    - Spring-Security
---

## Introduction
Spring Security is a customizable authentication and access service framework for server side Java-based enterprise software applications.

## Prerequisites
Before you begin this tutorial, you should:

1.  Be familiar working with STS or Eclipse.

### Setup: Exercise3-Starter
Import the source code from the downloaded folder Exercise3-Starter as given. Open STS, Select **File > Import > Maven > Existing Maven Projects** and select Exercise3-Starter folder.

  ![maven-import-step1](/images/spring_tutorial/import-maven-project-step1.png)

  ![maven-import-step2](/images/spring_tutorial/import-maven-project-step3.png)

## Adding Spring Security to the ExpenseReport App
This section will teach you how to integrate Spring Security into the ExpenseReport Application. These are the required dependencies to be added in your pom.xml:
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

In spring-security.xml, define the Web/HTTP security configuration by adding an HTTP tag, and define the URL pattern to be intercepted. Add the "jdbc-user service" tag, and define the query to get the user and authorities data from the database.

1.  To access the application, the user must be either `ROLE_USER` or `ROLE_MANAGER`. To validate this function, add `<intercept-url pattern="/" access="hasAnyRole('ROLE_USER','ROLE_MANAGER')"/>`.

2.  Since only the manager is allowed to approve expenses, restrict access to him alone by adding `<intercept-url pattern="/expensereports/approvals" access="hasRole('ROLE_MANAGER')"/>`.

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
        <intercept-url pattern="/expensereports/approvals" access="hasRole('ROLE_MANAGER')"/>
        <intercept-url pattern="/expensereports/changestate/**" access="hasRole('ROLE_MANAGER')"/>
        <intercept-url pattern="/" access="hasAnyRole('ROLE_USER','ROLE_MANAGER')"/>
        <intercept-url pattern="/**" access="isAuthenticated()" />
        <form-login login-page="/login" default-target-url="/expensereports"
            authentication-failure-url="/loginfailed" />
        <logout logout-success-url="/logout" />
        <session-management invalid-session-url="/sessiontimeout" >
            <concurrency-control max-sessions="1" />
        </session-management>
    </http>
    <beans:bean id="sas" class="org.springframework.security.web.authentication.session.SessionFixationProtectionStrategy" />

    <authentication-manager>
        <authentication-provider>
            <jdbc-user-service data-source-ref="dataSource"

           users-by-username-query="
              select username,password,enabled
              from users where username=?"

           authorities-by-username-query="
              select u.username, r.rolename from users u, role r
              where u.role_roleid = r.roleid and u.username =?  "

        />
        </authentication-provider>
    </authentication-manager>

</beans:beans>

```

To load this file as a part of configuration, add **@ImportResource("/WEB-INF/spring-security.xml")** in the `ComponentConfig` class.

Create `LoginController` in the `com.springsource.html5expense.contoller` package and add mapping for login,logout,loginfailed,signup. The login.jsp has two input elements for username and password(j_username and j_password). These are Spring's placeholder names for username and password, respectively.
When the form is submitted, it will be sent to the following action URL: j_spring_security_check.

Find the LoginController code [here](/frameworks/java/spring/tutorials/springmvc-jpa-postgres/code/logincontroller.html).

## Authentication Flow
* Once you have added Spring security into the Expense Reporting App, it will ensure that the user has been authenticated and authorized to access the resource. The below figure shows how the user interacts with the application.

    ![sequence-flow](/images/spring_tutorial/Spring-security-flow.png)

## Check Point
1. Let's build the app and test it locally. Right click on the app and select **Run As -> Maven clean**.

    ![maven_clean.png](/images/spring_tutorial/maven_clean.png)

2. Once Maven clean completes, select **Run As -> Maven install**. Dependencies will be downloaded from pom.xml.

    ![maven_install.png](/images/spring_tutorial/maven_install.png)

3. Select **Run As -> Maven build**.

    ![maven_build.png](/images/spring_tutorial/maven_build.png)

4. Use **tomcat:run** in Goals.

    ![maven_run.png](/images/spring_tutorial/maven_run.png)

5. Once the Server starts, open your browser and enter application url : `http://localhost:8080/html5expense/createNewExpense`. This time the browser will not open a form to create a new expense. You will be automatically redirected to the login page instead.

    ![STS Fabric Server.png](/images/spring_tutorial/localhost_login.png)

## Complete Application Code
If you are getting any errors, download the Exercise3-Complete code and import it into STS.

## Deploying to Cloud Foundry
* To learn more about deploying Spring Apps with PostgreSQL to Cloud Foundry, please refer [here](/frameworks/java/spring/tutorials/springmvc-jpa-postgres/springmvc-app-with-postgresql-deployment-to-cloudfoundry.html).

<a class="button-plain" style="padding: 3px 15px;" href="/frameworks/java/spring/tutorials/springmvc-jpa-postgres/spring-expensereport-app-tutorial.html">Prev</a>
