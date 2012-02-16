---
title: Box Integration
description: Sample Application Deployment via Gallery
tags:
    - 3rd-party-service
    - code-attached
    - boilerplate-sample
---

The [Box Ruby Sample app](https://github.com/cloudfoundry-samples/box-sample-ruby-app) has a redesigned interface for interacting with your content on Box.
It demonstrates usage of the main functions of the API, including file upload/download,
account tree viewing, file preview, and more.

Box has teamed up with CloudFoundry to offer direct deployment of this sample application the Box App Editing Wizard

## Getting Started
1. Register [here](http://www.box.net/developers/services) as a Box Developer
2. Create a first app or edit an existing one

## Deploying the Box Sample Application
1. From the Cloud Services menu Click the Cloud Foundry icon
  ![cloud_services.png](/images/screenshots/box/cloud_services.png)
2. Click "Create My App" in the popup
  ![box-cf-1.png](/images/screenshots/box/box-cf-1.png)
3. Click "Deploy my App"
  ![box-cf-2.png](/images/screenshots/box/box-cf-2.png)
4. Log into Cloud Foundry
  ![cf-login.png](/images/screenshots/box/cf-login.png)
5. Click orange deploy button to deploy box-sample-ruby-app
  ![box-deploy.png](/images/screenshots/box/box-deploy.png)
6. After 5 second window, Box login screen should appear
  ![box-cf-success.png](/images/screenshots/box/box-cf-success.png)

## Testing the Box Sample Application
1. Authorize your sample app to access your Box data by logging into Box
  ![box-sample-app-login.png](/images/screenshots/box/box-sample-app-login.png)

2. You should now see a web page with "Add Folder", "Add File" buttons.
3. Drill into your folder and confirm files you tossed in are available. Add another file or two.
  ![box-add-file.png](/images/screenshots/box/box-add-file.png)

## Making changes to how the app works

* Go to [box-sample-ruby-app](https://github.com/cloudfoundry-samples/box-sample-ruby-app) on GitHub
* Fork it to your own GitHub repo and clone it to your machine
* [Install VMC](/tools/vmc/installing-vmc.html) if you haven't already.
* Make changes to code and test locally following instructions on the README inside the box-sample-ruby-app repo
* Deploy changes to Cloud Foundry using `vmc update <app_name>`
* For more VMC commands see the [VMC Quick reference Guide](/tools/vmc/vmc-quick-ref.html)
