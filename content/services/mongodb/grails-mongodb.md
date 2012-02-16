---
title: Using MongoDB with Grails
description: Grails Development with the MongoDB Service
tags:
    - mongodb
    - grails
---

The [Grails](http://www.grails.org) framework allows rapid development of web applications that run in a Java servlet container. Grails is based on the Groovy language and the Spring framework.

To use Grails with Cloud Foundry, install the Cloud Foundry Grails plugin from your application directory:

```bash
$ grails install-plugin cloud-foundry
```

To use [MongoDB](http://www.mongodb.org/) in your Grails applciation, install the Grails [MongoDB plug-in](http://grails.org/plugin/mongodb).

```bash
$ grails install-plugin mongodb
```

Once you install the mongodb plugin, your application has access to MongoDB. The MongoDB plug-in allows you to map GORM entities to MongoDB Collections. It also provides a low-level API for directly accessing the MongoDB driver for Java.

## References

+	[Grails Cloud Foundry Plugin User Guide](http://grails-plugins.github.com/grails-cloud-foundry/)
+	[Gorm for Mongo](http://grails.github.com/inconsequential/mongo/manual/index.html) Reference Documentation