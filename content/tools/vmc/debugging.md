---
title: Debugging with VMC
description: Debugging Problems With Your Applications
tags:
    - debug
    - vmc
---

This chapter includes the following topics to help you debug problems with your application:

+ [Viewing Log Files](#viewing-log-files)
+ [Attaching a Debugger to Your Application](#attaching-a-debugger-to-your-application)

##Viewing Log Files

If you get errors when attempting to deploy, start, or run your application, it is helpful to view the various log files which will likely contain error codes or exceptions that can help you pinpoint what the problem is.

Use the `vmc files` command to get a list of available log files and to view recent entries in a particular log file.

For example, to get a list of all the log files related to the `hotels` application, and see which ones are non-zero in size and thus contain information, execute the following VMC commands (with sample output shown):

```bash

prompt$ vmc files hotels logs

  stderr.log                                2.2K
  stdout.log                                5.3K

prompt$ vmc files hotels tomcat/logs

  catalina.2011-10-18.log                   2.2K
  host-manager.2011-10-18.log                 0B
  localhost.2011-10-18.log                  247B
  manager.2011-10-18.log                      0B

```

To view the contents of the files in the `logs` directory (such as `stderr.log`), use the following command:

```bash
prompt$ vmc files hotels logs/stderr.log
```

To view the contents of the Tomcat log files (such as `catalina.2011-10-18.log`) file, use the following command:

```bash
prompt$ vmc files hotels tomcat/logs/catalina.2011-10-18.log
```

Both of these commands display the information to the terminal, so you must redirect the output to a file if you want to send it to support:

```bash
prompt$ vmc files hotels tomcat/logs/catalina.2011-10-18.log  > catalina.log
```

##Attaching a Debugger to Your Application

Documentation not yet available.
