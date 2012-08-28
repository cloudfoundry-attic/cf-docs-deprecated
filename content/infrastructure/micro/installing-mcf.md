---
title: Installing Micro Cloud Foundry
description: Get the Micro Cloud Foundry VM Installed and Running
tags:
    - mcf
---

This document helps you to download and get the Micro Cloud Foundry VM installed and
running. When you complete these tasks, you can begin publishing your
application to Micro Cloud Foundry.

**Subtopics**

+   [About Micro Cloud Foundry](#about-micro-cloud-foundry)
+   [Installation Overview](#installation-overview)
+   [Downloading the Micro Cloud Foundry Virtual Machine](#downloading-the-micro-cloud-foundry-virtual-machine)
+   [Starting and Configuring the Micro Cloud Foundry Virtual Machine](#starting-and-configuring-the-micro-cloud-foundry-virtual-machine)
+   [Registering a Micro Cloud Foundry User with vmc](#registering-a-micro-cloud-foundry-user-with-vmc)
+   [Next Steps](#next-steps)

## About Micro Cloud Foundry

Micro Cloud Foundry provides the VMware open platform as a service in a
standalone environment running in a virtual machine. It is a self-contained
local development environment as similar as possible to the Cloud Foundry cloud,
making application development for the cloud and the transition from
development to production much more seamless.

When you use Micro Cloud Foundry, you run a local virtual machine that provides
the same services Cloud Foundry provides your application. The virtual machine
connects with VMware servers to set up DNS for your application. You develop on
your local computer and then use the `vmc` Ruby command line utility, or the
Eclipse/Spring Tool Suite (STS) Cloud Foundry plug-in, to publish your
application to your Micro Cloud Foundry. You and others on your network, can
then test your application at
http://api.*appname*.cloudfoundry.me.

When you are ready to move the application to production, you use `vmc` or STS
to publish the application to a local or hosted Cloud Foundry instance.

## Installation Overview

Installing Micro Cloud Foundry includes downloading the virtual machine and then
running it in [VMware Workstation][1], [VMware Fusion][2] (Mac), or
[VMware Player][3] and configuring it. While you are at the Micro Cloud Foundry
Web site, you register a unique application name (*appname*) for your
application.

The Micro Cloud Foundry virtual machine connects to VMware servers
to set up DNS for your application. This is accomplished by using a
configuration token, which is generated for you when you visit the Micro Cloud
Foundry Web site at https://www.cloudfoundry.com/micro/dns. The generated DNS
token is good for one use; if your network changes, you must return to the Micro
Cloud Foundry Web site, generate a new token, and, in the Micro Cloud Foundry
virtual machine, select option 4 to reconfigure your domain.

Before you begin, be sure you have these items:

+   A [Cloud Foundry](http://cloudfoundry.com/) account.

+   [VMware Workstation][1], [VMware Fusion][2] (Mac), or [VMware Player][3] installed.

+   Ruby and the [vmc](/tools/vmc/installing-vmc.html) gem installed.

+   If you develop in Java, [Spring Tool Suite (STS)] or Eclipse with
the VMware Cloud Foundry plug-in installed. See [Configuring Spring Tool Suite or Eclipse for Cloud Foundry](/tools/STS/configuring-STS.html).

[1]: http://www.vmware.com/products/workstation/overview.html
[2]: http://www.vmware.com/products/fusion/overview.html
[3]: http://www.vmware.com/products/player/overview.html

## Downloading the Micro Cloud Foundry Virtual Machine

1.  In your Web browser, go to [Micro Cloud Foundry](https://cloudfoundry.com/micro).

2.  Click on 'Get Micro Cloud Foundry' and then log in with your Cloud Foundry email and password. (Click **Get an Account** if you need an account.)

2.  Check the box to accept the End User License Agreement, and click **Accept**.

3.  Enter a unique domain name for your Micro Cloud Foundry. The name you enter is checked in real-time so you can see if it is available.

	![Create Domain](/images/screenshots/installing-mcf/micro_dns.jpg "Micro DNS")

4.  Click **Create**.

    Your new domain name is reserved and a configuration token is created for you.

	![Domain Reserved](/images/screenshots/installing-mcf/micro_reserved.png "Domain Reserved")

5.  Write down the configuration token. You will need it in a later step.

6.  Click **Download Micro Cloud Foundry VM** and save the file.

## Starting and Configuring the Micro Cloud Foundry Virtual Machine

1.  Unzip/tar the compressed Micro Cloud Foundry VM. This creates the folder
    `micro` containing the virtual machine files.

2.  Start VMware Workstation, VMware Fusion, or VMware Player and open the `micro/micro.vmx` file.

3.  Power on the virtual machine.

4.  At the Welcome screen, select **1** to configure.

5.  Set a password for Micro Cloud Foundry by entering and confirming the new password.

6.  At the *Select network:* prompt, enter **1** to configure networking with DHCP.

7.  At the *HTTP proxy:* prompt, press Enter to choose **none** for HTTP proxy.

    If you are behind an HTTP proxy, enter the URL for the proxy server, for example `http://192.168.1.125:8023`.

8.  Enter the configuration token from the Micro Cloud Foundry Web site.

```bash
              Welcome to VMware Micro Cloud Foundry version 1.2.0

Network up
Micro Cloud Foundry not configured

1. configure
2. refresh console
3. help
4. shutdown VM

select option: 1

set password Micro Cloud Foundry VM user
Password: ********
Confirmation: ********
Password changed!

1. DHCP
2. Static
Select network: 1

HTTP proxy: |none|

Enter Micro Cloud Foundry configuration token or offline domain name: shock-throw-caption
```
The Micro Cloud Foundry virtual machine verifies your DNS and configures the Micro Cloud.

**Note**
: If you plan to use Micro Cloud Foundry without an Internet connection, enter a fictious domain name instead of the DNS configuration token. This is an advanced configuration option described in [Using Micro Cloud Foundry](/infrastructure/micro/using-mcf.html#working-offline-with-micro-cloud-foundry).


## Registering a Micro Cloud Foundry User with vmc

Registering a user creates a user account on the Micro Cloud Foundry virtual
machine. You log in with this account to publish and manage applications.

*Notes:*
: See [Installing the Command-Line Interface (vmc)](/tools/vmc/installing-vmc.html) for
    help installing `vmc`.

: See [VMC Quick Reference](/tools/vmc/vmc-quick-ref.html) for additional information
    about using the `vmc` command.

: See [Configuring Spring Tool Suite or Eclipse for Cloud
    Foundry](/tools/STS/configuring-STS.html) for help setting up the Cloud Foundry
    Integration plug-in in STS or Eclipse and for registering a Micro Cloud
    Foundry user from within the plug-in.

In the steps that follow, *appname* is the domain you registered for
your application at the Micro Cloud Foundry Web site.

Target your Micro Cloud Foundry. In a shell, enter the following command:

```bash
$ vmc target api.appname.cloudfoundry.me
```
Create a new account using the `vmc register` command:

```bash
$ vmc register
```

Enter your email address.

Enter a password and confirm it when requested.

```bash
$ vmc target api.pubs.cloudfoundry.me
Successfully targeted to [http://api.pubs.cloudfoundry.me]

$ vmc register
Email: myemail@mydomain.com
Password: ********
Verify Password: ********
Creating New User: OK
Successfully logged into [http://api.pubs.cloudfoundry.me]
```

You are now ready to log in with `vmc` or set up Spring Tool Suite to deploy your
applications to Micro Cloud Foundry.

## Next Steps

+ [Log in to your Micro Cloud Foundry with vmc](using-mcf.html#using-microcloud-foundry-vmc)
+ [Log in to your Micro Cloud Foundry with Spring Tool Suite (STS)](using-mcf.html#using-micro-cloud-foundry-sts)
+ [Day-to-day Micro Cloud Foundry Management](using-mcf.html)
