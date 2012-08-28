---
title: Get Started
description: Getting Started with Cloud Foundry
---

1. Register at the [Cloud Foundry Web page](http://www.cloudfoundry.com). You will receive an email with your user credentials.

2. Install one or both of the Cloud Foundry tools that you use to deploy and manage applications:

	+ [VMC](/tools/vmc/installing-vmc.html) if you prefer using command-line utilities.
	+ [Cloud Foundry extension to Spring Tool Suite (STS) or Eclipse](/tools/STS/configuring-STS.html) if you prefer using graphical IDEs.

3. Optionally install [Micro Cloud Foundry](/infrastructure/micro/installing-mcf.html) to develop on a version of Cloud Foundry that runs on your local computer.

4. [Configure your applications to deploy on Cloud Foundry.](frameworks.html)

    You may not need to perform any configuration at all. However, if your
    application requires multiple services or you want more control, then
    you need to do some minimal configuration.

5. [Deploy your application using VMC or STS.](/tools/deploying-apps.html)

    This step includes creating service instances, binding the service
    instances to your application, and managing the lifecycle of your
    deployed application.