---
title: Node.js Auto-reconfig FAQs
description: Node.js Auto-reconfig FAQs
tags:
    - faq
    - nodejs
---

### What are the limitations of Node.js Auto-reconfig?
Auto-reconfiguration of services work only under the following conditions:

* You are using only one service of the given type. For example, one mysql or one redis service.
* You are using service node module from the list of supported modules above, or any other that is using one of them for service connection underneath.
* Your application does not use cf-runtime or cf-autoconfig node modules.
* Your application is a typical Node.js application. For a complex application you may need to consider to opt-out of auto-reconfiguration and use cf-runtime node module described in the next blog post in these series.
Opting out of auto-reconfiguration


### Is Auto-reconfig turned-on by default?
Yes

### How to turn-off Auto-reconfiguration?

Auto-reconfiguration can be turned off by creating `cloudfoundry.json` file in application base folder with the option “cfAutoconfig” set as false.

```javascript

{ “cfAutoconfig” : false }
```
In addition, as mentioned above, auto-reconfiguration will not work if application is using cf-runtime node module.

### What version of Node.js is required?

Node.js v0.6.x, v0.8.x and above

### How to know when auto-reconfig doesn't work?

Typically you will see 'could not connect' to a given service. That's the clue. 

### What are the supported modules / services?

The following is the list of supported modules:

* [amqp](https://github.com/postwait/node-amqp)
* [mongodb](https://github.com/mongodb/node-mongodb-native)
* [mongoose](https://github.com/learnboost/mongoose)
* [mysql](https://github.com/felixge/node-mysql)
* [pg](https://github.com/brianc/node-postgres)
* [redis](https://github.com/mranney/node_redis)

According to [search.npmjs.org](search.npmjs.org) these modules are the most dependent on. This means there are many other modules that use these modules for database connection layer and auto-reconfiguration will be applied to them as well.



