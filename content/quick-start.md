---
title: Quick Start
description: A Hello World tutorial for Cloud Foundry
---

This tutorial takes you through the process of creating a simple Ruby application and deploying it on Cloud Foundry.

* Sign up for a Cloud Foundry account from the [Cloud Foundry Web page](http://www.cloudfoundry.com/). You will receive an email with your user credentials.

* If needed, install Ruby and Ruby Gems on your computer.

    See [Installing Ruby and RubyGems](/frameworks/ruby/installing-ruby.html).

* 	Install the vmc gem, using the following command:

```bash
$ sudo gem install vmc
```

	Note that `sudo` is not needed on all operating systems.

* This example uses Sinatra, so install the Sinatra gem:

```bash
$ sudo gem install sinatra
```

* 	Target the Cloud Foundry and log in using the credentials you recieved in email:

```bash
$ vmc target api.cloudfoundry.com
    $ vmc login

```

* The first time you log in, you should change your password:

```bash
$ vmc passwd
```

* Create a directory for your application and change into it.

```bash
$ mkdir my-first-app
	$ cd my-first-app
```

* Create a new file, `hello.rb`, with the following contents:

		require 'sinatra'
		get '/' do
			"Hello from Cloud Foundry"
		end

* Deploy your application using the `vmc push` command:

```bash
$ vmc push
```

    You can accept most defaults by pressing `Enter` at the prompts.  But be sure to enter a unique URL at the `Application Deployed URL` prompt. The name of the application needs to be unique only among your set of applications.

    The following is an example of interacting with the `vmc push` command:

        Would you like to deploy from the current directory? [Yn]
        Application Name: hello
        Application Deployed URL: 'hello.cloudfoundry.com'?  hello-bob.cloudfoundry.com
        Detected a Sinatra Application, is this correct? [Yn]
        Memory Reservation [Default:128M]  (64M, 128M, 256M, 512M or 1G) (Press Enter to take default)
        Would you like to bind any services to 'hello'? [yN]:

        Uploading Application:
          Checking for available resources: OK
          Packing application: OK
          Uploading (0K): OK
        Push Status: OK
        Staging Application: OK
        Starting Application: OK

* Invoke your application in a browser using the deployment URL you specified, for example, `http://hello-bob.cloudfoundry.com`.

# Next Steps

+ [Read more about Cloud Foundry](/infrastructure/overview.html)
+ [Install Micro Cloud Foundry on your local computer](/infrastructure/micro/installing-mcf.html)
+ [Install the Cloud Foundry Extension in STS or Eclipse](/tools/STS/configuring-STS.html)
+ [Configure your Spring applications to use Cloud Foundry services](/frameworks/java/spring/spring.html)
+ [Configure your Ruby on Rails applications to use Cloud Foundry services](/frameworks/ruby/rails.html)
+ [Deploy and manage applications with more complicated requirements](/tools/deploying-apps.html)
+ [Read the VMC Quick Reference guide](/tools/vmc/vmc-quick-ref.html)
