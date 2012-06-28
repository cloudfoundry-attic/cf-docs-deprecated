---
title: Creating a Simple Ruby Application
description: Create a Simple Ruby Application and Deploy to Cloud Foundry
tags:
    - ruby
    - sinatra
---

Follow these steps to create a simple Ruby application and deploy it to Cloud Foundry.

Before you begin, be sure you have met these prerequisites:

+ 	[Installed Ruby and RubyGems](installing-ruby.html)
+	[Installed VMC](/tools/vmc/installing-vmc.html)
+	[Registered for a Cloud Foundry account](https://my.cloudfoundry.com/signup)

This example creates a short program using the Sinatra Ruby framework.

Create a new directory, `hello`, and change into it.

```bash
$ mkdir hello
$ cd hello
```

In the `hello` directory, create the file `hello.rb` with the following contents:

```ruby
require 'rubygems'
require 'sinatra'
get '/' do
  "Hello from Cloud Foundry"
end

```

Install the Sinatra Ruby gem, since this application requires it:

```bash
$ gem install sinatra
```

Target Cloud Foundry and log in with your Cloud Foundry credentials:

```bash
$ vmc target api.cloudfoundry.com
	Successfully targeted to [http://api.cloudfoundry.com]

	$ vmc login
	Attempting login to [http://api.cloudfoundry.com]
	Email:me@example.com
	Password: *********
	Successfully logged into [http://api.cloudfoundry.com]

```

Push the application to Cloud Foundry. You can press `Enter` at most prompts to accept the default, but be sure to enter a unique URL for your application:

```bash
$ vmc push
	Would you like to deploy from the current directory? [Yn]: y
	Application Name: hello
	Application Deployed URL [hello.cloudfoundry.com]:
	Detected a Sinatra Application, is this correct? [Yn]:
	Memory Reservation (64M, 128M, 256M, 512M, 1G) [128M]:
	Creating Application: OK
	Would you like to bind any services to 'hello'? [yN]:
	Uploading Application:
	  Checking for available resources: OK
	  Packing application: OK
	  Uploading (0K): OK
	Push Status: OK
	Staging Application: OK
	Starting Application: OK

```

Use your browser to view the application at the specified URL.
