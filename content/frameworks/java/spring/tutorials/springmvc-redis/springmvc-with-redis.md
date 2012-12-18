---
title: Spring MVC with Redis
description: Spring MVC with Redis
tags:
    - spring
    - sts
    - redis
    - maven
    - tomcat
---
## Introduction
This is a guide for Java developers using the Spring framework and Redis key-value store to build and deploy their apps on Cloud Foundry. It shows you how to set up and successfully deploy Spring MVC application using Redis to Cloud Foundry.

## Prerequisites
Before you get started, you need the following:

+  A [Cloud Foundry account](http://cloudfoundry.com/signup).

+  The [vmc](/tools/vmc/installing-vmc.html) Cloud Foundry command line tool.

+  A [Spring Tool Suite™ (STS)](http://www.springsource.org/spring-tool-suite-download) installation.

+  A [Cloud Foundry plugin for STS](/tools/STS/configuring-STS.html).

+  A [Redis 2.4](http://www.redis.io/) installation.


## Overview
The Spring Framework provides a comprehensive programming and configuration model for modern Java-based enterprise applications - on any kind of deployment platform. A key element of Spring is infrastructural support at the application level: Spring focuses on the "plumbing" of enterprise applications so that teams can focus on application-level business logic, without unnecessary ties to specific deployment environments. For more details visit [spring homepage](http://www.springsource.org/spring-framework).

We are going to access the Redis key-value store using the [Spring Data Redis](http://www.springsource.org/spring-data/redis) which provides the integration with Redis.

<table class="spring-tutorial-index-table">
    <thead>
            <tr>
                <th>Exercise No</th>
                <th>Exercises</th>
                <th>Starters</th>
                <th>Complete</th>
            </tr>
    </thead>
    <tbody>
            <tr>
                <td>1</td>
                <td><a href='/frameworks/java/spring/tutorials/springmvc-redis/spring-expensereport-app-using-redis.html'>Create a Spring MVC Application using Redis</a></td>
                <td><a href='/code/tutorials/springmvc-redis/springmvc-expensereport-redis_starter.zip'>springmvc-expensereport-redis_starter.zip</a></td>
                <td><a href='/code/tutorials/springmvc-redis/springmvc-expensereport-redis_complete.zip'>springmvc-expensereport-redis_complete.zip</a></td>
            </tr>
    </tbody>
</table>

<a class="button-plain" style="padding: 3px 15px; float: right" href="/frameworks/java/spring/tutorials/springmvc-redis/spring-expensereport-app-using-redis.html">Next</a>
