---
title: Spring Insight
description: Using Spring Insight to trace Java applications
tags:
    - spring
    - performance-monitoring
---

Spring Insight is a Cloud Foundry service that gives you real-time visibility into your deployed applications.  Spring Insight graphs the health of your applications over time so that you can immediately see any performance problems in your application as they occur.

Spring Insight captures Web application events known as *traces*. A trace represents a thread of execution. It is usually started by an HTTP request but can also be started by a background job. A trace contains operations, which represent significant points in the execution of the trace, for example, a JDBC query or transaction commit. Insight uses trace data to calculate summary information and to lead you to the specifics of why your application is not performing as well as it should be.

For detailed information about using Spring Insight and how to interpret and filter the performance information it displays, see [Using Spring Insight](http://pubs.vmware.com:8080/vfabric5/topic/com.vmware.vfabric.tc-server.2.6/operations/using-browsing-resources.html).   Although that documentation specifically applies to Spring Insight running in a non-Cloud Foundry environment (such as a standalone tc Runtime instance), the usage of the **Browse Resources**, **Recent Activity**, and **Administration** tabs is similar.

**Subtopics**

+ [Installing Spring Insight In Your Cloud Foundry Account and Enabling Your Applications](#installing-spring-insight-in-your-cloud-foundry-account-and-enabling-your-applications)
+ [How Spring Insight Works With Cloud Foundry](#how-spring-insight-works-with-cloud-foundry)
+ [Spring Insight's Impact on Cloud Foundry Quotas](#spring-insights-impact-on-cloud-foundry-quotas)
+ [Configuring Application and Insight Memory Settings](#configuring-application-and-insight-memory-settings)

##Installing Spring Insight In Your Cloud Foundry Account and Enabling Your Applications

*  In your browser, go to the [Spring Insight for Cloud Foundry](http://insight.cloudfoundry.com/) Web site and log in using your Cloud Foundry credentials.

    The first time you login, Spring Insight automatically installs itself into your Cloud Foundry account and creates and binds to a RabbitMQ service instance.  This takes a few seconds.  When the installation is complete, you will be redirected to your personal Spring Insight dashboard which lists all the applications you have deployed to Cloud Foundry:

    ![insight-cf-app.png](/images/screenshots/spring-insight/insight-cf-app.png "insight dashboard")

    When you subsequently browse and login to the [Spring Insight for Cloud Foundry](http://insight.cloudfoundry.com/) Web site, you will be automatically redirected to your dashboard.

    The **Cloud Foundry > Applications** screen lists your deployed applications; by default, Spring Insight is disabled for each one.

*  To enable Insight for an application, click on its name in the list on the left, then click `enable` in the right frame (see #2 in the image above.)   In the background, Insight installs and associates an agent with your application.

    Currently, only Java, Spring, and Grails Web applications can be enabled for Spring Insight.  You will know whether one of your deployed applications can be enabled for Spring Insight because it will have a leaf icon next to it in the list (see #1 in the image above.)

*  Click the **Cloud Foundry > Environment** tab to view information about the available and provisioned service instances in your Cloud Foundry account.  Spring Insight itself is bound to a provisioned RabbitMQ service instance, as shown below:

    ![insight-cf-env.png](/images/screenshots/spring-insight/insight-cf-env.png "insight dashboard environment")

*  To start actually using Spring Insight to monitor the performance of your applications, click the **Browse Resources** tab.  This screen shows the overall health of your application.  Note that only applications that have been enabled for Insight, and have had some recent activity, will show up in the list on the left.

    Click the **Recent Activity** tab to view the recent traces associated with your applications into which you can drill down.

For detailed information about using Spring Insight and how to interpret and filter the performance information it displays, see [Using Spring Insight](http://pubs.vmware.com:8080/vfabric5/topic/com.vmware.vfabric.tc-server.2.6/operations/using-browsing-resources.html).

##How Spring Insight Works With Cloud Foundry

When you install Spring Insight in your Cloud Foundry Account, Insight shows up as a deployed application, just like any other application you might have previously deployed.  Its name is `Insight-<id>`, where `id` is a unique identifier.  You can manage the Insight application just like any other, including deleting it with the `vmc delete` command.  This has the effect of deleting your dashboard; you can recreate it using the procedure above.

When you enable one of your deployed application for Spring Insight, Insight binds itself to your application as if it were a service, and shows up accordingly when you execute `vmc apps`.  For example:

```bash
$prompt vmc apps

+----------------+----+---------+---------------------------------+-----------------+
| Application    | #  | Health  | URLS                            | Services        |
+----------------+----+---------+---------------------------------+-----------------+
| Insight-18fe44 | 1  | RUNNING | insight-18fe44.cloudfoundry.com | Insight-18fe441 |
| hello          | 1  | RUNNING | hello-js.cloudfoundry.com       |                 |
| hotels         | 1  | RUNNING | hotels-js.cloudfoundry.com      | Insight-18fe441 |
+----------------+----+---------+---------------------------------+-----------------+

```

The Insight application uses a RabbitMQ service instance, which you can also see by executing `vmc services`. For example:

```bash
$prompt vmc services

============== System Services ==============

+------------+---------+---------------------------------------+
| Service    | Version | Description                           |
+------------+---------+---------------------------------------+
| mongodb    | 2.0     | MongoDB NoSQL store                   |
| mysql      | 5.1     | MySQL database service                |
| postgresql | 9.0     | vFabric Postgres database service     |
| rabbitmq   | 2.4     | RabbitMQ messaging service            |
| redis      | 2.2     | Redis key-value store service         |
+------------+---------+---------------------------------------+

=========== Provisioned Services ============

+-----------------+----------+
| Name            | Service  |
+-----------------+----------+
| Insight-18fe441 | rabbitmq |
+-----------------+----------+

```

## Spring Insight's Impact on Cloud Foundry Quotas

The Insight dashboard is a Web application installed on your Cloud Foundry account.  With respect to your quota, it counts as one application and uses 512 MB of memory.

Spring Insight also provisions a RabbitMQ service instance that the Insight agents use to communicate with the Insight dashboard. The RabbitMQ service counts as one service of your Cloud Foundry quota.

Finally, each application that you have enabled for Insight will require additional memory. While this varies with the load and instrumentation, Cloud Foundry recommends that when you deploy an application that you want to monitor with Insight, you should use at least 256 MB of your memory quota.

##Configuring Application and Insight Memory Settings

When determining how much memory your application is consuming, Cloud Foundry uses the corresponding process' RSS (resident set size).  If the RSS grows beyond your application's configured memory threshold, Cloud Foundry will stop or kill it. This has special implications in Java, because an application's maximum heap usage is often well below the RSS.

To avoid a scenario in which the RSS is exceeded without hitting the maximum heap usage, increase the memory allocated to your application, or decrease its heap size.

For instance, if your application runs with around 256 MB, but does not hit that limit, use the following `vmc` commands to change change the memory settings:

```bash
prompt$ vmc mem my-application 300m
prompt$ vmc env-add my-application INSIGHT_OVERRIDES="-Xmx200m"
```

Note that this modification of memory settings is not specific to Insight; Cloud Foundry will kill regular Java applications that do not hit their max heap.  But Insight-enabled applications are more likely to hit these limits due to the reliance on JVM datastructures outside of the heap, on PermGen and in the code cache.
