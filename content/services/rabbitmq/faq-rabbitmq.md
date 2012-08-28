---
title: RabbitMQ FAQ
description: RabbitMQ FAQ
tags:
    - rabbitmq
    - faq
---

### What is the RabbitMQ service?
The RabbitMQ service on Cloud Foundry brings the messaging functionality of RabbitMQ to developers building applications on Cloud Foundry. Like the rest of CloudFoundry.com, the RabbitMQ service is currently offered as a free beta service.

### What is RabbitMQ?
RabbitMQ is a popular open-source [message broker](http://en.wikipedia.org/wiki/Message_broker), developed at VMware. See the [RabbitMQ site](http://www.rabbitmq.com/) for more details.

### How does the RabbitMQ service relate to Cloud Foundry?
In addition to providing a platform for applications, Cloud Foundry provides a portfolio of on-demand services that developers can bind to their applications. Services available at the launch of Cloud Foundry included MySQL, MongoDB and Redis. The RabbitMQ service is on a par with those other services. You use the standard Cloud Foundry vmc commands (or their Spring Tool Suite equivalents) to create, bind, unbind and delete RabbitMQ services.

### Which languages and frameworks can I use with the RabbitMQ service?
Any that are supported by Cloud Foundry and that have an AMQP client library available.

We have developed applications for Cloud Foundry using the following language, framework and client library combinations:

Java Spring applications with [Spring AMQP](http://www.springsource.org/spring-amqp) (version 1.0.0.RC2)
Ruby on Rails and Ruby Sinatra applications with the [bunny gem](https://github.com/ruby-amqp/bunny) (version 0.7.4)
Node.js applications using with [node-amqp](https://github.com/postwait/node-amqp) (version 0.1.0)

### How do I get started?
At [github.com/rabbitmq/rabbitmq-cloudfoundry-samples](https://github.com/rabbitmq/rabbitmq-cloudfoundry-samples) you can find introductory samples showing how to use the service from various languages and frameworks, including Spring Java, Ruby on Rails, Ruby Sinatra, and node.js. And there are detailed tutorials for [Rails](http://www.rabbitmq.com/cloudfoundry/rails) and [Spring](http://www.rabbitmq.com/cloudfoundry/spring) on this site.

You can find more information about how to develop applications using RabbitMQ, which also applies to the RabbitMQ service, on the [RabbitMQ site](http://www.rabbitmq.com/getstarted.html).

### Which protocols can I use to access the RabbitMQ service?
Currently the service supports the core protocols of RabbitMQ: [AMQP](http://en.wikipedia.org/wiki/AMQP) versions 0-8 and 0-9-1. In the future we plan to incorporate other protocols supported by RabbitMQ plugins. Let us know if you are interested in support for a particular protocol.

### Is the RabbitMQ service accessible to applications outside of CloudFoundry.com?
Not at present.

### Which of the RabbitMQ use cases listed on the [Getting Started](https://www.rabbitmq.com/getstarted.html) page on RabbitMQ.com work with the RabbitMQ service?
All these use cases will work.

### Does the RabbitMQ service support clustering?
Not yet, but we are considering this for the future.

### Which version of rabbitmq-server is the RabbitMQ service based on?
Currently rabbitmq-server-2.4.1.

### How is the RabbitMQ service priced?
Like the rest of Cloud Foundry, it is initially offered as a free beta service.

### Is RabbitMQ now part of the Cloud Foundry open source project?
No, the RabbitMQ service is not a part of the Cloud Foundry open source project.

### Do I need to update vmc to use the RabbitMQ service?
No, you do not need to update vmc to use the RabbitMQ service. But note that vmc continues to be enhanced, so you should occasionally update it with the rubygems command gem update vmc

### Where can I ask questions or send feedback about the service?
Please ask your questions on the forums at [support.cloudfoundry.com](http://support.cloudfoundry.com) or on [StackOverflow](http://stackoverflow.com/questions/tagged/cloudfoundry).