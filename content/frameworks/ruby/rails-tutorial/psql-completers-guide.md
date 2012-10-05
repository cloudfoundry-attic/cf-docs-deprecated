---
title: Expense Reporting Application with Rails
description: Application Setup from Complete Code
tags:
    - ruby
    - rails
---

##Introduction

This tutorial will teach you how to setup the application from the downloaded source code after completion of respective chapters.

+ Ruby-1.9.2-p180 should be installed on your machine via `rvm`.

+ [PostgreSQL](http://www.postgresql.org/download/) should be installed on your machine.

+ As there is `.rvmrc` file in application code, automatically it will use the particular ruby version along with gemset.

+ Install the gems specified in `Gemfile` by running the following command.

```bash
$ bundle install
```

+ Modify the authentication details in database configuration file to connect to PostgreSQL database. Do this by replacing `<username>` and `<password>` in `config/database.yml` file with actual values.

+ Run the following commands to setup the database.

```bash
$ bundle exec rake db:create
$ bundle exec rake db:migrate
$ bundle exec rake db:seed # only for chapter-3
```

+ Its done. Now go back to the respective <a href="javascript:javascript:history.go(-1)">tutorial</a> check point and check out the application as mentioned there.