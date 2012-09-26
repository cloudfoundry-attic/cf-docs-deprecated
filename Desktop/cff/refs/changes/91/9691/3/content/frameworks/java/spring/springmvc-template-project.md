---
title: Hello world Spring MVC Project
description: Spring MVC Template Project
tags:
    - Spring MVC
    - STS
    - tomcat
    - maven
---

## Introduction
This section provides details on how to get started with STS (SpringSource Tool Suite) and Spring MVC to create a Spring MVC application to display Hello world message.

## Create Spring MVC Template Project
Please open your STS and select dashboard by clicking,
    **Help -> Dashboard**

From the dashboard view select **Spring Template Project**. Window shown below opens up.

![Spring MVC Template Project](/images/spring_tutorial/spring_template_project_mvc.png)


Choose **Spring MVC Project**.
The first time you do it, STS will ask you to download some extra elements from the Internet.

![spring_template_project_mvc_download.png](/images/spring_tutorial/spring_template_project_mvc_download.png)

You’ll need to give your project name as `expensereport` and top-level package as `com.springsource.html5expense`


Click on **Finish** – and STS will create the project.

Before run the project,select **Run As -> Maven clean**

![maven_clean.png](/images/spring_tutorial/maven_clean.png)

Once Maven clean completes, select **Run As -> Maven install**. Dependencies will be downloaded from pom.xml.

![maven_install.png](/images/spring_tutorial/maven_install.png)

## Troubleshoot
After this step if you have any errors,
 Check whether you are behind proxy server. If then change your maven `settings.xml` to point your proxy server,

```xml
<settings>
  <proxies>
   <proxy>
      <active>true</active>
      <protocol>http</protocol>
      <host>proxy.somewhere.com</host>
      <port>8080</port>
      <username>proxyuser</username>
      <password>somepassword</password>
      <nonProxyHosts>www.google.com|*.somewhere.com</nonProxyHosts>
    </proxy>
  </proxies>
</settings>
```

Now Select **Run As -> Maven build**

![maven_build.png](/images/spring_tutorial/maven_build.png)

And give Maven goal as , **tomcat:run**

![maven_run.png](/images/spring_tutorial/maven_run.png)

## Check Point
Once server start is completed, open your browser and go to our application url `http://localhost:8080/html5expense` it will open the home page for the project which displays 
Hello world message with the current server time.

![Hello World Output](/images/spring_tutorial/hello_world.png)

[<==Prev       ](/frameworks/java/spring/spring-getting-started-with-STS.html)                         [                 Next==>](/frameworks/java/spring/spring-expensereport-app-tutorial.html)
