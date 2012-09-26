---
title: Hello world Spring MVC Project
description: Create Spring MVC Hello world Application
tags:
    - Spring MVC
    - STS
    - tomcat
    - maven
---

## Introduction
This section provides details on how to get started with STS (SpringSource Tool Suite) and Spring MVC to create a Spring MVC application to display Hello world message.

## Create Spring MVC Template Project
1.  Please open your STS and select dashboard by clicking, **Help -> Dashboard**. From the dashboard
view select **Spring Template Project**. Window shown below opens up.

	![Spring MVC Template Project](/images/spring_tutorial/spring_template_project_mvc.png)

     Fig 1.1


2.  Choose **Spring MVC Project**. The first time you do it, STS will ask you to download some extra
elements from the Internet.

	![spring_template_project_mvc_download.png](/images/spring_tutorial/spring_template_project_mvc_download.png)

     Fig 1.2

3.  You’ll need to give your project name as `expensereport` and top-level package as `com.springsource.html5expense`

4.  Click on **Finish** – and STS will create the project.

5.  Before run the project, select **Run As -> Maven clean**

	![maven_clean.png](/images/spring_tutorial/maven_clean.png)

     Fig 1.3

6.  Once Maven clean completes, select **Run As -> Maven install**. Dependencies will be downloaded from pom.xml.

	![maven_install.png](/images/spring_tutorial/maven_install.png)

     Fig 1.4

7.  Now Select **Run As -> Maven build**

	![maven_build.png](/images/spring_tutorial/maven_build.png)

     Fig 1.5

8.  And give Maven goal as, **tomcat:run**

	![maven_run.png](/images/spring_tutorial/maven_run.png)

     Fig 1.6

## Check Point
*  Once server start is completed, open your browser and go to the application url `http://localhost:8080/html5expense` it will open the home page for the project which displays Hello world message with the current server time.

	![Hello World Output](/images/spring_tutorial/hello_world.png)

[![Prev](/images/spring_tutorial/prev_doc.png)](/frameworks/java/spring/tutorials/springmvc-jpa-postgres/spring-getting-started-with-STS.html)  <span style="float: right;">[![Next](/images/spring_tutorial/next_doc.png)](/frameworks/java/spring/tutorials/springmvc-jpa-postgres/spring-expensereport-app-tutorial.html)</span>
