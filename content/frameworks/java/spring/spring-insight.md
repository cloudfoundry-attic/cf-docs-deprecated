---
title: Spring Insight on Cloud Foundry FAQ
description: Monitoring Java Applications on Cloud Foundry with Spring Insight
tags:
    - spring insight
    - faq
---

**Q. What is Insight?**
Insight is a byte-code instrumentation monitoring tool for development and production. It is designed to provide light-weight visibility into the operation of a user's application and enable applications to more easily transition into production. Insight for Cloud Foundry is in the beta phase and open to those who have been given an access token.

**Q. Who do I contact with problems?**
Please submit questions and issues to the Cloud Foundry support site: [http://support.cloudfoundry.com](http://support.cloudfoundry.com/)

**Q. What does it mean to have Insight on Cloud Foundry?**
Once the Insight Dashboard is installed, it can be used to enable code-level monitoring on any Java application installed within the user's Cloud Foundry account.

**Q. How do I start monitoring a Java application on Cloud Foundry?**
After installing a dashboard through [http://insight.cloudfoundry.com](http://insight.cloudfoundry.com/) you should be redirected to the dashboard. All of your applications should be listed on the dashboard Cloud Foundry tab (1). Those that can be instrumented with Insight will have a little Spring leaf icon(2). On these applications, you can enable Insight through the details panel (3). Monitored data will appear under your Application on the Browse Resources tab (4).

**Q. What is Insight-[randomid].cloudfoundry.com?**
This is your personal dashboard. All data sent from your applications is sent to this application. Since it is visible as just another application from vmc, you can even uninstall the dashboard the way you would uninstall a normal Cloud Foundry application.

**Q. Which Application frameworks are supported?**
We currently support Java, Spring, and Grails web applications.

**Q. Will using the service have any impact on my Cloud Foundry Application?**
When activating Insight, Cloud Foundry installs an agent on the Tomcat that's running your app. Our tests show that this agent has negligible impact on performance and only increases response time by a millisecond.

**Q. How does Insight impact customer Cloud Foundry Quotas?**
The Insight dashboard is a web application installed on your account. It will take 512MB of your quota. In addition we provision a RabbitMQ service that's used by the Insight agents to communicate with the Insight dashboard. The Insight dashboard counts as one application and the RabbitMQ service counts as one service of your account. Each application which is monitored will require additional memory. While this varies with the load and instrumentation, we recommend deploying applications with at least 256MB of memory quota.

**Q. Who is the targeted end user?**
The target end users are developers who are interested in making sure that their application is behaving as expected in Cloud Foundry and need a tool to help them resolve problems in real-time.

**Q. When will Insight be generally available?**
We announced the private beta of Insight for Cloud Foundry at SpringOne 2GX (Oct 2011). We plan to announce a public beta a few weeks after SpringOne, but not before we are confident that everything is working as it should.

**Q. How can I use Insight with my own plugins outside of Cloud Foundry?**
Insight is available for free download with tc Server Developer Edition or Spring Tool Suite: [http://springsource.org/insight](http://springsource.org/insight)

**Q. How do new users gain access to Insight on Cloud Foundry?**
To gain access to Insight on Cloud Foundry, you will need to be a registered user of Cloud Foundry. While in beta, you will also need a token. We distribute tokens once in a while when we're ready to increase our beta test group.

**Q. How does Spring Insight work?**
Insight uses aspect-oriented programming to create an additional instrumentation layer around select methods within the user's application. The data from these aspects is passed to a secondary webapp which compresses, filters, and forwards the data to the Insight Dashboard where it can be viewed.

**Q. How much additional memory must be allocated to an Application when it is monitored by Insight?**
Cloud Foundry sets the minimum and maximum heap of a web application to the configured application memory, but uses the resident size (RSS) of the process to verify compliance. This method does not account for PermGen usage or general process overhead through native libraries or memory-mapped files. Since Insight uses more PermGen because of the class generation through AspectJ we recommend a minimum of 256MB for an application, depending on the complexity. Additionally we require the installation of the Insight Dashboard which uses 512MB of memory.

**Q. My Application is running out of memory and Cloud Foundry is killing it. How do I configure the memory settings?**
You may need to alter your Application's memory settings to avoid having Cloud Foundry kill your Application. Cloud Foundry uses the process' RSS (resident set size) to determine how much memory your Application is consuming, and if the RSS grows beyond your application's configured memory threshold, Cloud Foundry will kill it. This has special implications in Java, because an application's -Xmx (maximum heap usage) is often well below the RSS.

To avoid a scenario in which the RSS is exceeded without hitting the maximum heap usage, increase the memory allocated to your application, or decrease its heap size.

For instance, if your application runs with 256MB (but does not hit that limit):

```bash
  $  vmc mem my-application 300m
  $  vmc env-add my-application INSIGHT_OVERRIDES="-Xmx200m"
```

Note that this modification of memory settings is not specific to Insight; Cloud Foundry will kill regular Java applications which do not hit their max heap but Insight-instrumented applications are more likely to hit these limits due to reliance on JVM datastructures outside of the heap, in PermGen, and in the code cache.

**Q. My application reports that there is a duplicate RabbitServiceInfo bean, how do I work around this?**
The RabbitServiceInfo is created for each rabbit entry in the VCAP_SERVICES list. That list would include the application rabbit service and the service for Insight. With some configurations this could result in a duplicate spring bean. An alternative is to wire the RabbitServiceInfo explicitly with a unique id.