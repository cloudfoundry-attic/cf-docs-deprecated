---
title: Using VMC in Non Interactive Mode
description: How to Use vmc in Non Interactive Mode
tags:
    - vmc
    - non interactive
---

VMC client's default mode of operation is 'Interactive'.

It is possible to invoke VMC in non-interactive mode by using one of the following command line switches.

```bash
    -n, --no-prompt
        --noprompt
        --non-interactive
```

For example : VMC in interactive mode : (clicked 'return' to take the default values)

```bash
$ vmc push hello
Would you like to deploy from the current directory? [Yn]:
Application Deployed URL: 'hello.vcap.me'?
Detected a Sinatra Application, is this correct? [Yn]:
Memory Reservation [Default:128M] (64M, 128M, 256M, 512M, 1G or 2G)
Creating Application: OK
Would you like to bind any services to 'hello'? [yN]:
Uploading Application:
  Checking for available resources: OK
  Packing application: OK
  Uploading (0K): OK
Push Status: OK
Staging Application: OK
Starting Application: OK
```

VMC in non-interactive mode

```bash
$ vmc push hello -n
Creating Application: OK
Uploading Application:
  Checking for available resources: OK
  Packing application: OK
  Uploading (0K): OK
Push Status: OK
Staging Application: OK
Starting Application: OK
```

For any of the interactive questions, if non-default value for the question is needed, that can be passed in the command line.  (For example --mem for memory).

For a list of all the options supported by VMC, please try

```bash
$ vmc help options
```