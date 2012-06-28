---
title: Debugging Applications with Cloud Foundry Integration for Eclipse
description: Debugging Applications with Cloud Foundry Integration for Eclipse
tags:
    - eclipse
    - sts
---

## Debugging an Application

   Applications deployed to Micro Cloud Foundry or local cloud with debugging support also enable debugging functionality like starting an application
   in debug mode, or connecting an application already running in debug mode to the Eclipse debugger.

   ![Debug from Editor](/images/screenshots/configuring-STS/cf_eclipse_editor_debug.png)

   For applications running in debug mode, applications can be restarted or update and restarted in either **Debug** or **Run** mode,
   through a drop down menu beside both the **Restart** and **Update and Restart** buttons.
   The button icon and hover tooltip text indicate which mode the application is currently running in.

   ![Debug Update and Restart](/images/screenshots/configuring-STS/cf_eclipse_editor_updaterestart_debug.png)

   If an application is already running in debug mode, but not connected to the local debugger, the editor allows a user to
   connect to the debugger without restarting the application through a **Connect to Debugger** button.

   ![Connect to Debugger](/images/screenshots/configuring-STS/cf_eclipse_editor_connect_to_debugger.png)

   If an application is disconnected from the local debugger in the Eclipse Debug view, it will still continue running in
   debug mode in the Cloud Foundry target until it is stopped or removed from the Cloud Foundry target.

   Debugging behavior in the Cloud Foundry Integration is similar to debugging local applications, and it is integrated
   into the Eclipse Debug perspective. Users can set break points, step through code, and suspend an application.

   ![Application Debugging](/images/screenshots/configuring-STS/cf_eclipse_debugging_app.png)

   Multiple applications, and application instances of the same application, can be debugged at the same time, with the Debug view differentiating each debugged application instance by the Cloud Foundry target
   name, application name, and instance ID.