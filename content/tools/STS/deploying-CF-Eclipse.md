---
title: Deploying Applications using Cloud Foundry Integration for Eclipse
description: Deploying Applications and Binding Services using Cloud Foundry Integration for Eclipse
tags:
    - eclipse
    - sts
---

**Subtopics**

+   [Define a New Server for a Cloud Foundry Target](#define-a-new-server-for-a-cloud-foundry-target)
+   [Deploying Applications to Cloud Foundry from STS or Eclipse](#deploying-applications-to-cloud-foundry-from-sts-or-eclipse)
    +   [Define application services](#define-application-services)
    +   [Bind application services](#bind-application-services)
+   [Update and Restart an Application](#update-and-restart-an-application)

When deploying applications to Cloud Foundry using Cloud Foundry Integration for Eclipse, a Cloud Foundry target must first be defined.
Once defined, it will appear in the Eclipse Servers View, where users can drag and drop applications from the Eclipse Project Explorer
to deploy them to the selected Cloud Foundry target.

## Define a New Server for a Cloud Foundry Target

In STS or Eclipse, you define a new Server to represent a Cloud Foundry target
and then deploy applications to it.

Follow these steps to define the new Server.

*  Choose **Window > Show View > Servers**.

*  Either click new server wizard or right-click on an empty area in the Servers view and choose **New > Server**.

    ![STS New Server](/images/screenshots/configuring-STS/cf_eclipse_empty_servers_view.png)

    The **Define a New Server** wizard starts.

*  Expand the VMware folder, select Cloud Foundry.

    ![STS New Server Wizard](/images/screenshots/configuring-STS/cf_eclipse_new_cf_server.png)

*  Specify a display name for the Cloud Foundry server instance that should be created in Server name. Server host name should
   remain localhost. Click **Next**.

   ![STS New Server Wizard Account](/images/screenshots/configuring-STS/cf_eclipse_new_server_account.png)

*  Select the Cloud Foundry target you want to set up from the URL list.

    +   **VMware Cloud Foundry**. The VMware-hosted open platform as a service.

    +   **Local Cloud**. A local installation of the VMware Cloud Application Platform (VCAP).

    +   **Microcloud**. A Micro Cloud Foundry virtual machine.

*  If you selected **Microcloud**, enter the domain name you registered for
   your Micro Cloud Foundry at http://cloudfoundry.com/micro and a descriptive name.

    ![Create Microcloud Target](/images/screenshots/configuring-STS/sts-mcf-domain-name.png)

*  Click **OK**.

*  Enter an email address and password for the Cloud Foundry target.

    + For a VMware Cloud Foundry target, the email must be
    pre-registered at the Web site. A **CloudFoundry.com** signup button allows users to pre-register an e-mail account.

    + For Local Cloud or Microcloud targets, use an email address you have
    already registered, or enter a new email and password to register, and then click
    **Register Account...**.

    Click **Validate Account** to test whether the account is valid on the
    target Cloud Foundry.

* Click **Finish**.

    The new Cloud Foundry target appears in the Servers view.

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
    Servers view.

    ![Show Servers](/images/screenshots/configuring-STS/cf_eclipse_servers_view_cf_server.png)

    Currently deployed applications, if any, are listed beneath the servers.

*  To deploy an application, drag it to the target Cloud Foundry Server in the Servers view.

    ![Drag and Drop App](/images/screenshots/configuring-STS/cf_eclipse_drag_drop_app.png)

    Alternatively, you can double click on the server and the Cloud Foundry editor will open, allowing users to drag and drop
    applications into the Applications tab.

    The Cloud Foundry Integration Extension examines the application to determine the application type and verify that it is
    deployable to the selected Cloud Foundry server. If so, it opens an Application details wizard, where the application can be
    configured and optional services bound. Supported application types include **Spring**, **Grails**, **Lift**, and **Java Web**.

    ![Application details](/images/screenshots/configuring-STS/cf_eclipse_application_details.png)

*  Change the name of the application, if desired, and select the correct
    Application Type if the integration extension incorrectly identified the
    application's framework, then click **Next**.

    **Note:**
    The application name, here, is only used to identify the
    application for administration. The name exposed to users in the
    application's URL is set on the next page of the wizard.

    ![Launch deployment](/images/screenshots/configuring-STS/cf_eclipse_application_details_regular_start.png)

*  Edit the Deployed URL and change the Memory Reservation if needed.

    The Deployed URL must be unique at the target Cloud Foundry.

*  If you need to bind services to the application, click **Next** to bind them prior to deployment, or unselect **Start application on
   deployment** and bind the services after deployment through the Cloud Foundry server editor.

*  Click **Finish**, although optionally a user may select **Next** to bind services prior to deployment.

    The application is deployed. If you chose **Start application on deployment** it is started and can be accessed at the mapped
    URL. If deploying to a microcloud or local cloud with debugging support, an additional option for starting the application in
    **Debug** mode is displayed.

    ![Launch deployment debug](/images/screenshots/configuring-STS/cf_eclipse_application_details_debug_start.png)

*  If clicking **Next**, existing services can be bound to the application, or additional services can be defined and then bound.

    ![Bind services on deployment](/images/screenshots/configuring-STS/cf_eclipse_application_deployment_services.png)

*  Once deployed, in the Servers view, double-click the application to open the editor and display the application stats as well as
   controls to start, stop, restart, update and restart the application, and also change the application's configuration and bound services.

    ![Applications Control Panel](/images/screenshots/configuring-STS/cf_eclipse_cf_editor.png)

## Define Application Services

You must define a service before you bind it to a deployed application. The
Cloud Foundry Integration extension retrieves a catalog of available services
from the target Cloud Foundry. After you define a service, you can bind it to
the application. Services can be bound to an application from the Application details
wizard during deployment or from the Cloud Foundry server editor after deployment, if the application is stopped.

Follow these steps to define a service from the editor.

*  In the Servers view, double-click the application name.

    ![STS Applications Tab](/images/screenshots/configuring-STS/cf_eclipse_cf_editor.png)

    The Applications tab shows details about the applications on the Cloud
    Foundry target.

*  In the Services section, click the **Add service** icon.

    ![STS Add Service](/images/screenshots/configuring-STS/cf_eclipse_editor_services_table.png)

*  Provide a name for the new service and select the type of service.

    ![STS Service Configuration](/images/screenshots/configuring-STS/sts-service-configuration.png)

    The **Type** list contains all of the service types available on the
    target Cloud Foundry.

*  Click **Finish**.

    The plug-in requests the service from Cloud Foundry and the new service
    appears in the Services section.

## Bind Application Services

When you bind a service to an application, the Cloud Foundry Integration
extension updates the application configuration files to access the defined
service. The application must not be running when you bind services.

*  If it is running, stop the application:

   + In the Servers view, right-click on the application name and select **Stop**, or
   + In the Applications panel in the Cloud Foundry server editor, select the application and click the **Stop** button.

*  In the Applications panel, select the application you want to bind a
    service to.

*  Select the service you want to bind in the Services panel and drag it to the
    Application Services Panel.

    ![sts bind service](/images/screenshots/configuring-STS/cf_eclipse_bind_service.png)

*  Click the **Start** button.


## Update and Restart an Application

The Applications Tab in the Cloud Foundry editor allows users to modify application details like memory, number of running instances, mapped application URLs, as well as starting, stopping, restarting, and update and restarting an application.


   ![Regular Update and Restart](/images/screenshots/configuring-STS/cf_eclipse_editor_regular_updaterestart.png)

   Users can restart an application without publishing changes in an application through a **Restart** button or context menu action
   in the Servers view, or update changes in a deployed application and restarting it using an **Update and Restart** option.

   **Update and Restart** incrementally publishes local changes in an application, and is optimized to only push resources that have
   changed since the last publish.

   Stop and Starting an application performs a full publish.

