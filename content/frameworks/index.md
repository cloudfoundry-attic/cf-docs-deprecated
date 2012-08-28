---
title: Overview
description: Configuring Applications to Run on Cloud Foundry
---

Cloud Foundry supports the following application development frameworks:

+ Spring
+ Ruby on Rails
+ Ruby and Sinatra
+ Node.js
+ Grails

The support includes a runtime environment that enables your applications to execute on Cloud Foundry and deployment tools (vmc and STS) that detect the framework and automate configuration and deployment to Cloud Foundry. You may not need to do anything special to deploy to Cloud Foundry. If the application requires services, such as a database, and you follow the frameworks' conventions, vmc or STS can handle needed configuration changes when the application is deployed.
There are instances, however, when your application will need minimal configuration, in particular if it requires multiple services.

Each application framework has its own configuration process, described in the following sections:

+ [Spring Applications](/frameworks/java/spring/spring.html)
+ [Grails Applications](/frameworks/java/spring/grails.html)
+ [Node.js Applications](/frameworks/nodejs/nodejs.html)
+ [Ruby, Rails and Sinatra](/frameworks/ruby/ruby-rails-sinatra.html)

For a list of available Services see [Services](/services.html).

After you configure your application to integrate with Cloud Foundry services, you use standard Cloud Foundry tools (VMC, Spring Tool Suite, or the Eclipse plugin) to create instances of these services and bind them to your applications when you deploy them to Cloud Foundry. See [Deploying Applications](/tools/deploying-apps.html) for details on how to use these tools.
