---
title: Accessing Cloud Foundry Services
description: Tunneling to a Cloud Foundry Service with Caldecott
tags:
    - debug
    - tunneling
    - vmc
---

Application services such as databases and messaging are built-in to Cloud Foundry, which helps speed development and ease application management. You do not have to install and manage servers because they are part of the Cloud Foundry platform. Your application code interacts with the Cloud Foundry services, but sometimes you need to access a service interactively. For example, you might want to run ad hoc queries, view data to help debug a problem, or import or export data.

An application called Caldecott, after the Caldecott Tunnel in the Berkeley Hills in California, allows you to access your services in the cloud. Caldecott uses tunneling to connect a port on your local computer to the service in the cloud. The `vmc tunnel` command uploads the Caldecott application to your Cloud Foundry instance, sets up the tunneling, and offers to start a standard client on your computer to work with the service.

## Prerequisites

+	Caldecott requires Ruby 1.9.2.

+ 	You must have vmc 0.3.14 or later. Use the following command to check your version:

```bash
$ vmc -v
```

	Enter this vmc command to update the vmc gem:

```bash
$ gem update vmc
```

+ 	Caldecott can start a client program for the service you want to access, for example `mysql` for MySQL or `psql` for PostgreSQL. The client must be installed on your computer and be on the execution PATH so that Caldecott scripts can start it. If you want to use a different client, Caldecott will display the connection information and credentials you will need to connect to the service. The following table shows the client programs Caldecott can start for each service.

<table class="std">
	<tr>
		<th>Service</th>
		<th>Client</th>
	</tr>
	<tr>
		<td>MongoDB</td>
		<td><tt>mongo</tt></td>
	</tr>
	<tr>
		<td>MySQL</td>
		<td><tt>mysql</tt></td>
	</tr>
	<tr>
		<td>PostgreSQL</td>
		<td><tt>psql</tt></td>
	</tr>
	<tr>
		<td>rabbitmq</td>
		<td><i>none</i></td>
	</tr>
	<tr>
		<td>Redis</td>
		<td><tt>redis-cli</tt></td>
	</tr>
</table>

## Setting up Caldecott

Install the Caldecott Ruby gem with this command:

```bash
$ gem install caldecott
```

Note
: The Caldecott gem currently requires the eventmachine gem, which requires native compilation. We are working to simplify this.

## Using Caldecott

* Target the Cloud Foundry instance and log in with your Cloud Foundry credentials:

```bash
$ vmc target api.cloudfoundry.com
$ vmc login

```

* Use this command to view the existing services:

```bash
$ vmc services
```

	The command output has two parts. The first, System Services, reports services that can be provisioned in the Cloud Foundry instance. You are interested in the second section, Provisioned Services, which lists your existing services.

		=========== Provisioned Services ============

	    +------------------+------------+
	    | Name             | Service    |
	    +------------------+------------+
	    | mongodb-12345    | mongodb    |
	    | mysql-12345      | mysql      |
	    | postgresql-12345 | postgresql |
	    | redis-12345      | redis      |
	    +------------------+------------+

* Create a tunnel to the service. For example, to connect to the MySQL service:

```bash
$ vmc tunnel mysql-12345
	Trying to deploy Application: 'caldecott'.
	Create a password:

```

	The first time you create a tunnel, vmc uploads the Caldecott application to your Cloud Foundry instance and prompts you to set a password to protect it. You will need to provide that password each time you create a tunnel to that Cloud Foundry instance in the future.

	After you provide the password, the Caldecott application is uploaded:

```bash
Uploading Application:
      Checking for available resources: OK
      Processing resources: OK
      Packing application: OK
      Uploading (1K): OK
    Push Status: OK
    Binding Service [mysql-12345]: OK
    Staging Application: OK
    Starting Application: OK

```

	See [More About the Tunnel Command](#more-about-the-tunnel-command) for a complete description of the `vmc tunnel` command options.

* Caldecott creates the tunnel and prompts you to start a client. Here is an example of a session using `mysql` to access data on Cloud Foundry:

```bash

	Getting tunnel connection info: OK

	Service connection info:
	  username : um4rwWyhwa07B
	  password : pBiBlqjINB6cm
	  name     : dd368741dbc1945cfb62315565efcf1b5

	Starting tunnel to mysql-24e9c on port 10000.
	1: none
	2: mysql
	Which client would you like to start?: 2
	Launching 'mysql --protocol=TCP --host=localhost --port=10000
	--user=um4rwWyhwa07B --password=pBiBlqjINB6cmdd368741dbc1945cfb62315565efcf1b5'

	Welcome to the MySQL monitor.  Commands end with ; or \g.
	Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
		.
		.
		.
	mysql>

```

* When you exit the client, the tunnel is disconnected.

	If you chose the `1: none` option, or if Caldecott does not have a default client for the service, start your preferred client in another window and leave this terminal window undisturbed until you are ready to close the tunnel. Then press `Ctrl-C` to exit vmc when you are done.

## More About the Tunnel Command

You can just enter the `vmc tunnel` command and respond to prompts to create a tunnel. The command allows you to select from a list of existing provisioned services, so you do not need to know the service name ahead of time.

The full syntax of the `vmc tunnel` command is:

	vmc tunnel [<servicename>] [--port <portnumber>] [<clientcmd>]

The `<servicename>` argument is the name of the service, as shown by the `vmc services` command. If you exclude `<servicename>`, vmc provides a list of existing services from which to choose.

The `<portnumber>` parameter is the port to use on the local machine. If omitted, vmc chooses an available port in the 10000 range.

The `<clientcmd>` argument is the name of the client program to start. Only the client names shown in the table in the [Prerequisites](#prerequisites) section are supported for this argument.
