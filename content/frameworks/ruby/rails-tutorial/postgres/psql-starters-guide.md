---
title: Expense Report Application with Rails and PostgreSQL
description: Application Setup from Starters Code
tags:
    - ruby
    - rails
---

##Introduction

This tutorial will teach you how to setup the application from the downloaded source code before proceeding to the rest of the chapters.

+ Ruby-1.9.2-p180 should be installed on your machine via `rvm`.

+ [PostgreSQL](http://www.postgresql.org/download/) should be installed on your machine.

+ Because of the '.rvmrc' file in the application code, the particular ruby version with the gemset will be automatically used.

+ Install the gems specified in `Gemfile` by running the following command.

```bash
$ bundle install
```

+ Modify the authentication details in the database configaration file to connect to PostgreSQL database. Do this by replacing `<username>` and `<password>` in `config/database.yml` file with actual values.

+ Run the following commands to setup the database.

```bash
$ bundle exec rake db:create
$ bundle exec rake db:migrate
$ bundle exec rake db:seed # only for chapter-4
```

+ If you get an error like `ExecJS::RuntimeUnavailable` while executing the above commands, then modify the below line in `Gemfile` from

```ruby
# gem 'therubyracer'
```

to

```ruby
gem 'therubyracer'
```

+ Now run the following command to install the gem and then execute the above mentioned commands to setup the database.

```bash
$ bundle install
```

+ That is all. Now follow the <a href="javascript:javascript:history.go(-1)">tutorial</a> to continue.