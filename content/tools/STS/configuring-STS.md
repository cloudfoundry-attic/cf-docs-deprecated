---
title: Installing Cloud Foundry Integration for Eclipse
description: Installing the Cloud Foundry Integration Extension for Eclipse or STS
tags:
    - eclipse
    - sts
---

If you are developing in Java with the Eclipse IDE or STS (Spring Tool Suite), install the Cloud Foundry Integration Extension to deploy applications to Cloud Foundry. This document describes how to install the extension and how to get started deploying applications from Eclipse or STS.

**Subtopics**

+   [Prerequisites](#prerequisites)
+   [Installing the Cloud Foundry Integration Extension in STS](#installing-the-cloud-foundry-integration-extension-in-sts)
+   [Installing the Cloud Foundry Integration Extension in Eclipse](#installing-the-cloud-foundry-integration-extension-in-eclipse)

## Prerequisites

You must have Eclipse or STS installed. You can download installers from the
following Web sites:

+   [Eclipse download](http://www.eclipse.org/downloads/)

    Confirm that you have **Eclipse IDE for JEE Developers** so that all known dependencies are satisfied.
    (In Eclipse, click **Help > About Eclipse** to view the version of Eclipse you have installed.).
    The minimum supported Eclipse JEE package is **Indigo**.

+   [Spring Tool Suite download](http://www.springsource.org/sts)

    Install version 2.9.0 or higher.

+   Cloud Foundry Integration for Eclipse 1.1.0 is currently the latest release of the integration, and is the first version to be open sourced under the
    Eclipse Public License (EPL). As a consequence, earlier versions of Cloud Foundry Integration, including those with version number 2.7.0,
    cannot be upgraded to the latest version, 1.1.0. Users must first uninstall older Cloud Foundry integrations prior to installing the latest
    version.

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

    ![STS Cloud Foundry install wizard](/images/screenshots/configuring-STS/cf_eclipse_install_wizard.png)

*  Cloud Foundry offers two install options:
   +  **Core / Cloud Foundry Integration**

      Required installation for users who wish to use the feature but not extend the Cloud Foundry Integration.

   + **Resources / Cloud Foundry Integration**

      Optional SDK installation of Cloud Foundry Integration source files for users who wish to extend the Cloud Foundry Integration.
      Users are not required to install **Core / Cloud Foundry** installation with this option.

*  When the installer completes, click **Yes** to restart STS or Eclipse.

## Installing the Cloud Foundry Integration Extension in Eclipse

In Eclipse, follow these steps to install the Cloud Foundry Integration Extension.

*  Select **Help > Eclipse Marketplace**.

    A panel opens showing plug-ins and add-ons.

*  In the **Find** field, enter "cloud foundry" and click **Go**.

    ![Eclipse Extension installation](/images/screenshots/configuring-STS/cf_eclipse_marketplace.png)

*  In the search results, select "Cloud Foundry Integration for Eclipse" and click **Install**.

    Eclipse examines resources and dependencies.

*  Click **Next** to begin the installation.

    An install wizard guides you through license acceptance and installation steps.

*  When the installation is complete, restart Eclipse.

    You are ready to connect to a Cloud Foundry cloud.

## Next Steps

+ [Deploying Applications and Binding Services](/tools/STS/deploying-CF-Eclipse.html)
+ [Debugging](/tools/STS/debugging-CF-Eclipse.html)
+ [Remote File Access](/tools/STS/remote-CF-Eclipse.html)

