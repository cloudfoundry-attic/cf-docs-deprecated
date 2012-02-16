---
title: Using RabbitMQ with Ruby
description: Ruby Development with the RabbitMQ Service
tags:
    - ruby
    - rabbitmq
    - tutorial
    - code-attached
---

Cloud Foundry supports [RabbitMQ](http://www.rabbitmq.com/), the
open-source message broker, as a service. The Cloud Foundry RabbitMQ service is
based on rabbitmq-server-2.4.1.

For more information about RabbitMQ, see these resources:

-   [Download](http://www.rabbitmq.com/download.html),
    [install](http://www.rabbitmq.com/install.html), and
    [configure](http://www.rabbitmq.com/configure.html) RabbitMQ.

-   [RabbitMQ Tutorials](http://www.rabbitmq.com/getstarted.html) cover
    the basics of creating messaging in your applications.

## Language and Framework Support
Languages and frameworks supported by Cloud
Foundry that have an Advanced Message Queue Protocol (AMQP) client library are
also supported by the RabbitMQ service. Ruby on Rails and Sinatra have been
deployed on Cloud Foundry with the [bunny gem](https://github.com/ruby-amqp/bunny), version 0.7.4.

## Protocol Support
The Cloud Foundry RabbitMQ service supports the core
protocols of RabbitMQ: AMQP versions 0-8 and 0-9-1. Other protocols will be
supported by RabbitMQ plugins.

Follow these steps to provision and bind RabbitMQ to an application:

*  Create the RabbitMQ service and bind it to your application:

```bash
$ vmc create-service rabbitmq --bind appname
```

*  Check the application:

```bash
$ vmc apps
```

Returns a list of applications on your cloud, with any associated
services.

## Rails and RabbitMQ

The RabbitMQ service is accessed through the [AMQP
protocol](http://www.amqp.org/) (versions 0.8 and 0.9.1), so your
application requires an AMQP client library to use the service.

A popular AMQP client libraries for Rails is
[bunny](https://github.com/ruby-amqp/bunny).  we will use it demonstrate
the process you would follow:

Add the bunny and json gems to your Gemfile.  (json is used to parse the service connection data):

``` ruby
gem 'bunny'
gem 'json'
```

Run bundle install to install the gems you added:

```bash
$ bundle install
```

Require the gems in your controller:

``` ruby
require 'bunny'
require 'json'
```

Update the controller class to get the connection string for the service and make the connection:

``` ruby
# Extracts the connection string for the rabbitmq service from the
# service information provided by Cloud Foundry in an environment
# variable.
def self.amqp_url
    services = JSON.parse(ENV['VCAP_SERVICES'], :symbolize_names => true)
    url = services.values.map do |srvs|
    srvs.map do |srv|
        if srv[:label] =~ /^rabbitmq-/
            srv[:credentials][:url]
            else
            []
        end
    end
    end.flatten!.first
end

# Opens a client connection to the RabbitMQ service, if one is not
# already open.  This is a class method because a new instance of
# the controller class will be created upon each request.  But AMQP
# connections can be long-lived, so we would like to re-use the
# connection across many requests.
def self.client
    unless @client
        c = Bunny.new(amqp_url)
        c.start
        @client = c
    end
    @client
end
```

Set up message queues in the controller:

``` ruby
# Return the "nameless exchange", pre-defined by AMQP as a means to
# send messages to specific queues.  Again, we use a class method to
# share this across requests.
def self.nameless_exchange
    @nameless_exchange ||= client.exchange('')
end

# Return a queue named "messages".  This will create the queue on
# the server, if it did not already exist.  Again, we use a class
# method to share this across requests.
def self.messages_queue
    @messages_queue ||= client.queue("messages")
end
```

Add controller methods to read and write messages:

``` ruby
# The action for our publish form.
def publish
    # Send the message from the form's input box to the "messages"
    # queue, via the nameless exchange.  The name of the queue to
    # publish to is specified in the routing key.
    HomeController.nameless_exchange.publish params[:message],
                                   :key => "messages"
    # Notify the user that we published.
    flash[:published] = true
    redirect_to home_index_path
end

def get
    # Synchronously get a message from the queue
    msg = HomeController.messages_queue.pop
    # Show the user what we got
    flash[:got] = msg[:payload]
    redirect_to home_index_path
end
```