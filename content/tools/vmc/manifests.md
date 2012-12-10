---
title: Manifests
description: Automating deployment and configuration with manifests
tags:
    - vmc
    - manifests
---

Manifests are `.yml` files, typically named `manifest.yml`. They contain
a description of your application, including things like their name,
framework, runtime, etc. They can be used to describe complex multi-app
hierarchies, or simply for automating the deployment of a small application.

## Getting Started

A basic manifest looks like this:

```
applications:
- name: hello-sinatra
  path: .
  framework: sinatra
  runtime: ruby19
  memory: 128M
  instances: 1
```

With this file existing as `manifest.yml` in the root of a basic Sinatra
application, `vmc push` will automatically create the application, push the
bits, and start it up with no interaction necessary.

Without a manifest, `vmc push` will ask if you want to save your configuration
afterwards. Otherwise, you can hand-write it.

The manifest is also used to make most application-centric commands slightly
easier to use, by making the name optional. For example, `vmc stats` will use
the manifest to determine the app name.


## Getting Complex

A multi-app manifest is basically the same as a single app manifest, but with
more entries under `applications`. You can also specify that apps depend on
one another, so VMC will start/restart them in the proper order.

For a multi-app manifest, commands like `stats` will treat the entire
hierarchy as a single application, thus getting the stats for each
sub-application.

```
applications:
  sender:
    name: sender-app
    path: ./sender
    framework: sinatra
    runtime: ruby19
    depends-on: receiver
  receiver:
    name: receiver-app
    path: ./receiver
    framework: sinatra
    runtime: ruby19
```

Here we've given each application a tag, rather than simply using a list. This
makes it easier to refer to them with `depends-on`. Here, the tags are
`sender` and `receiver`.

Note that the paths are relative to the manifest file. In this case, the
expected directory structure is:

```
./manifest.yml
./sender/{Gemfile,main.rb,...}
./receiver/{Gemfile,main.rb,...}
```

Because the paths are relative to the manifest file, you can refer to
arbitrary manifests and have things like deployments work as expected, for
example `vmc push -m ~/Projects/myapp/manifest.yml`.

You can also have the same path in the manifest twice, as long as the tags are
different.


## Symbol Resolution

Manifests also have support for symbol resolution, so you can use this to keep
the repetition low:

```
---
path: .
runtime: ruby19
framework: standalone
instances: 1
memory: 128M
url: ${name}.${target-base}

base-command: bundle exec ruby main.rb

applications:
- name: my-app-foo
  command: ${base-command} foo
- name: my-app-bar
  command: ${base-command} bar
  memory: 256M
```

This will result in the following applications:

```
hello-sinatra(master*): vmc apps
Getting applications... OK

name         status    usage      runtime   url
my-app-bar   running   1 x 256M   ruby19    my-app-bar.cloudfoundry.com
my-app-foo   running   1 x 128M   ruby19    my-app-bar.cloudfoundry.com
```

Note that `target-base` is a predefined symbol; it resolves to the base of the
target URL, so if your target is `api.cloudfoundry.com`, this will be
`cloudfoundry.com`.
