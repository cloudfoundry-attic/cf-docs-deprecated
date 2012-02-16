---
title: Configuring STS
description: Installing the Cloud Foundry Integration Extension for STS and Eclipse
tags:
    - sts
---

If you are developing in Java with the Eclipse IDE or STS
(SpringSource Tool Suite), install the Cloud Foundry Integration
Extension to deploy applications to Cloud Foundry. This document
describes how to install the extension and how to get started
deploying applications from Eclipse or STS.

**Subtopics**

+   [Prerequisites](#prerequisites)
+   [Installing the Cloud Foundry Integration Extension in STS](#installing-the-cloud-foundry-integration-extension-in-sts)
+   [Installing the Cloud Foundry Integration Extension in Eclipse](#installing-the-cloud-foundry-integration-extension-in-eclipse)
+   [Define a New Server for a Cloud Foundry Target](#define-a-new-server-for-a-cloud-foundry-target)
+   [Deploying Applications to Cloud Foundry from STS or Eclipse](#deploying-applications-to-cloud-foundry-from-sts-or-eclipse)
    +   [Provisioning Application Services](#provisioning-application-services)
    +   [Binding Services to an Application](#binding-services-to-an-application)
+   [Remote File Access](#remote-file-access)

## Prerequisites

You must have Eclipse or STS installed. You can download installers from the
following Web sites:

+   [Eclipse download](http://www.eclipse.org/downloads/)

    Confirm that you have **Eclipse IDE for JEE Developers** so that all known
    dependencies are satisfied. (In Eclipse, click **Help > About Eclipse** to
    view the version of Eclipse you have installed.)

+   [SpringSource Tool Suite download](http://www.springsource.com/downloads/sts)

    Install version 2.6.1 or higher.

## Installing the Cloud Foundry Integration Extension in STS

In STS, follow these steps to install the Cloud Foundry Integration Extension.

*  Select **Help > Dashboard**.

    The Dashboard opens.

*  Click the **Extensions** tab.

    STS loads a list of extensions.

*  Scroll down to the **Server and Clouds** category and select **Cloud Foundry Integration**.

    ![STS Server and Clouds extensions](/images/screenshots/configuring-STS/sts-cf-extension.png)

*  Click **Install**.

    An install wizard guides you through the installation steps.

    ![STS Cloud Foundry install wizard](/images/screenshots/configuring-STS/sts-cf-install-wiz.png)

*  When the installer completes, click **Yes** to restart STS.

*  Select **Window > Show View > Servers**.

    ![STS Servers Panel](/images/screenshots/configuring-STS/sts-servers-panel.png)

## Installing the Cloud Foundry Integration Extension in Eclipse

In Eclipse, follow these steps to install the Cloud Foundry Integration Extension.

*  Select **Help > Eclipse Marketplace**.

    A panel opens showing plug-ins and add-ons.

*  In the **Find** field, enter "cloud foundry" and click **Go**.

    ![Eclipse Extension installation](/images/screenshots/configuring-STS/eclipse_mkt.png)

*  In the search results, select "Cloud Foundry Integration" for your Eclipse
    version and click **Install**.

    Eclipse examines resources and dependencies.

*  Click **Next** to being the installation.

    An install wizard guides you through license acceptance and installation steps.

*  When the installation is complete, restart Eclipse.

    You are ready to connect to a Cloud Foundry cloud.

## Define a New Server for a Cloud Foundry Target

In STS or Eclipse, you define a new Server to represent a Cloud Foundry target
and then deploy applications to it.

Follow these steps to define the new Server.

*  Choose **Window > Show View > Servers**.

*  Right-click in the Servers panel and choose **New > Server**.

    ![STS New Server](/images/screenshots/configuring-STS/sts-new-server.png)

    The **Define a New Server** wizard starts.

*  Expand the VMware folder, select Cloud Foundry, and click **Next**.

    ![STS New Server Wizard](/images/screenshots/configuring-STS/sts-new-server-wizard.png)

*  Select the Cloud Foundry target you want to set up from the URL list.

    +   **VMware Cloud Foundry**. The VMware-hosted open platform as a service.

    +   **Local Cloud**. A local installation of the VMware Cloud Application Platform (VCAP).

    +   **Microcloud**. A Micro Cloud Foundry virtual machine.

    +   **vmforce**. The open Java cloud from VMware and salesforce.com.

*  If you selected **Microcloud**, enter the domain name you registered for
your Micro Cloud Foundry at http://cloudfoundry.com/micro and a descriptive name.

    ![Create Microcloud Target](/images/screenshots/configuring-STS/sts-mcf-domain-name.png)

*  Click **OK**.

*  Enter an email address and password for the Cloud Foundry target.

    + For VMware Cloud Foundry or vmforce targets, the email must be
    pre-registered at the Web site.

    + For Local Cloud or Microcloud targets, use an email address you have
    already registered, or enter a new email and password to register, and then click
    **Register Account...**.

    Click **Validate Account** to test whether the account is valid on the
    target Cloud Foundry.

* Click **Finish**.

    The new Cloud Foundry target appears in the Servers panel.

## Deploying Applications to Cloud Foundry From STS or Eclipse

The Cloud Foundry Integration Extension uses the Eclipse Web Tools Project (WTP)
Server infrastructure, which is used to deploy Java web applications to remote
servers.

There are three steps to deploy an application.

+   [Define application details](#define-application-details)
+   [Define application services](#define-application-services)
+   [Bind application services](#bind-application-services)

The latter two steps are only required when the application uses Cloud Foundry
services, such as a MySQL or vFabric Postgres database, or RabbitMQ messaging.

## Define Application Details

*  In STS or Eclipse, select **Windows > Show View > Servers** to display the
    Servers pane.

    ![Show Servers](/images/screenshots/configuring-STS/sts-show-servers.png)

    Currently deployed applications, if any, are listed beneath the servers.

*  To deploy an application, drag it to the target Cloud Foundry Server.

    Alternatively, you can right click the target Cloud Foundry Server, choose
    **Add and Remove...** and use the Add and Remove selector dialog to choose
    available applications to add to the Cloud Foundry server.

    The Cloud Foundry Integration Extension examines the application to
    determine the application type, then displays the Application details
    wizard.

    ![Application details](/images/screenshots/configuring-STS/sts-application-details.png)

*  Change the name of the application, if desired, and select the correct
    Application Type if the integration extension incorrectly identified the
    application's framework, then click **Next**.

    **Note:**
    The application name, here, is only used to identify the
    application for administration. The name exposed to users in the
    application's URL is set on the next page of the wizard.

    ![Launch deployment](/images/screenshots/configuring-STS/sts-launch-deployment.png)

*  Edit the Deployed URL and change the Memory Reservation if needed.

    The Deployed URL must be unique at the target Cloud Foundry.

*  If you need to bind services to the application, unselect **Start application on deployment**.

*  Click **Finish**.

    The application is deployed. If you chose **Start application on deployment** it is started and can be accessed at the mapped URL.

*  Double-click the application under the target server to check its status.

    ![Applications Control Panel](/images/screenshots/configuring-STS/sts-application-control-panel.png)

## Define Application Services

You must define a service before you bind it to a deployed application. The
Cloud Foundry Integration extension retrieves a catalog of available services
from the target Cloud Foundry. After you define a service, you can bind it to
the application.

Follow these steps to define a service.

*  In the Servers panel, double-click the application name.

    STS displays the Package Explorer in the navigation panel and the
    Applications tab in the workspace.

    ![STS Applications Tab](/images/screenshots/configuring-STS/sts-applications-tab.png)

    The Applications tab shows details about the applications on the Cloud
    Foundry target.

*  In the Services section, click the **Add service** icon.

    ![STS Add Service](/images/screenshots/configuring-STS/sts-add-service-button.png)

*  Provide a name for the new service and select the type of service.

    ![STS Service Configuration](/images/screenshots/configuring-STS/sts-service-configuration.png)

    The **Type** list contains all of the service types available on the
    target Cloud Foundry.

*  Click **Finish**.

    ![STS Service Defined](/images/screenshots/configuring-STS/sts-services-list.png)

    The plug-in requests the service from Cloud Foundry and the new service
    appears in the Services section.

## Binding Application Services

When you bind a service to an application, the Cloud Foundry Integration
extension updates the application configuration files to access the defined
service. The application must not be running when you bind services.

*  If it is running, stop the application:

        + In the Server panel, right-click on the application name and select
        **Stop**, or

        + In the Applications panel, select the application and click the
        **Stop** button.

*  In the Applications panel, select the application you want to bind a
    service to.

*  Select the service you want to bind in the Services panel and drag it to the
    Application Services Panel.

    ![sts bind service](/images/screenshots/configuring-STS/sts-bind-service.png)

*  Click the **Start** button.

## Remote File Access

The Cloud Foundry Integration extension provides a file browser you can use to
view files deployed to Cloud Foundry. For example, you can view configuration
files and log files on the remote Cloud Foundry server.

Follow these steps to access the Remote Systems view.

*  Select **Window > Show View > Servers**.

*  In the Servers panel, expand a server and double-click on a deployed application.

*  Under the Application Details pane, click **Remote Systems View**.

    ![Remote Systems View](/images/screenshots/configuring-STS/remote-systems-view-link.png)

    The Remote Systems tab appears below the workspace.

*  Browse the file tree and open files. Right-click a file to see the options
    that are available to you.

	![Browsing Files in Remote Files view](/images/screenshots/configuring-STS/sts-browsing-files.png)
