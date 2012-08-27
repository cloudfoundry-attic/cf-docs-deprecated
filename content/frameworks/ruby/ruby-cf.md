---
title: Ruby, Rails and Cloud Foundry
description: Things to Know about Ruby, Rails and Cloud Foundry
tags:
    - ruby
    - rails
    - bundler
    - Gemfile
    - git
---

This article provides useful tips when building Ruby Apps on Cloud Foundry.
Many of the initial limitations have now been removed.
To review the latest news about Ruby support on Cloud Foundry visit:
[http://blog.cloudfoundry.com/tag/ruby/](http://blog.cloudfoundry.com/tag/ruby/)

Here are the highlights of the topics covered below:

+ Your application MUST supply a `Gemfile` that specifies ALL required gems
+ Pre-package your app to the greatest possible extent
+ Ruby 1.8 and Ruby 1.9 are supported


## Gems and Libraries

Some important comments about Gems and Gemfiles:

GEM list completeness - The proper way to interact with cloud foundry is to supply a complete `Gemfile` and
then also run `bundle install;bundle package` each time you touch your Gemfile and
prior to any `vmc push` or `vmc update` commands.

The supported way of using gems is via [Bundler](http://gembundler.com/)
VCAP **does not support** alternative method to Bundler [Isolate](https://github.com/jbarnette/isolate).

**Your application MUST supply a `Gemfile` that specifies ALL required gems as well as a `Gemfile.lock` with the specific versions of the gems **

For more information on `bundle package` see [http://gembundler.com/bundle_package.html](http://gembundler.com/bundle_package.html)


## Bundler

In General – Watch out for complicated and confusing bundle settings.
While VCAP tries to handle your configuration as best it can, it is possible to
write Gemfiles that cannot be satisfied, such as requesting an old version of Rails and a brand-new version of Rack.
These combinations fail immediately; you won't need to wait to push your app to
the cloud to see it breaks. In the same token, Gemfiles and Gems that are too vague about versions can still create as well.

Along with published gems, Gemfiles can refer to git repos by url, branch name, etc. And Bundler will build them using a specialized mechanism.
**VCAP now supports reading Gemfiles with git urls.**

Example `Gemfile`:

``` ruby

source :rubygems

# Gets the json gem from rubygems
gem "json", "~> 1.4.6"

# Gets the vcap_services_base gem from cloudfoundry's vcap-services-base repo on github.com at master branch
gem 'vcap_services_base', :git => 'git://github.com/cloudfoundry/vcap-services-base.git'

# Gets the vcap_logging gem from cloudfoundry's common repo on github.com at the specified ref on the master branch
gem 'vcap_logging', :require => ['vcap/logging'], :git => 'git://github.com/cloudfoundry/common.git', :ref => 'b96ec1192'

# Gets the eventmachine from cloudfoundry eventmachine repo on github.com using the release-0.12.11-cf branch
gem 'eventmachine', :git => 'git://github.com/cloudfoundry/eventmachine.git', :branch => 'release-0.12.11-cf'

```


You can only link to gems using paths if the paths are relative inside your app and the gems in the path have been built.

+ gem :path => "some/path"

Note that both patforms and git urls are now supported. This allows you to work in a Windows environment for example and
require additional gems which will not be installed on Cloud Foundry when your app starts running.

``` ruby

gem 'vcap_common', :require => ['vcap/common', 'vcap/component'], :git => 'git://github.com/cloudfoundry/vcap-common.git'
gem 'vcap_logging', :require => ['vcap/logging'], :git => 'git://github.com/cloudfoundry/common.git', :ref => 'b96ec1192'
gem 'vcap_services_base', :git => 'git://github.com/cloudfoundry/vcap-services-base.git'
gem 'warden-client', :require => ['warden/client'], :git => 'git://github.com/cloudfoundry/warden.git'

group :test do
  gem "webmock"
  gem "rake"
  gem "rack-test"
  gem "rspec"
  gem "simplecov"
  gem "simplecov-rcov"
  gem "ci_reporter"
end

```

Library dependency resolution - When a library you require has dependencies,
you need to pay special attention to the way it tries to load dependent library versions (through $LOAD_PATH).

### Bundler groups

Groups are fully supported allowing you to specify `development` and `test` groups for example and then exclude them for your app.

Example `Gemfile`

``` ruby

gem "sinatra"
gem "newrelic_rpm"

group :development do
  gem "vmc"
end

group :test do
  gem "webmock"
  gem "rake"
  gem "rack-test"
  gem "rspec"
  gem "simplecov"
  gem "simplecov-rcov"
  gem "ci_reporter"
end

```

Note that you can now set environment variable `BUNDLE_WITHOUT` to specify excluding development and test
Also note that Cloud Foundry will now honor the values you set in `RAILS_ENV` and `RACK_ENV`

``` bash

vmc push --nostart

vmc env-add <app_name> RAILS_ENV=staging
vmc env-add <app_name> BUNDLE_WITHOUT=test:development

vmc start <app_name>

```

### Platforms

Platforms are now fully supported, allowing you to require different gems for different OSs
Here is an example `Gemfile` using platforms:

``` ruby

# Unix Rubies (OSX, Linux)
platform :ruby do
  gem 'rb-inotify'
end

# Windows Rubies (RubyInstaller)
platforms :mswin, :mingw do
  gem 'eventmachine-win32'
  gem 'win32-changenotify'
  gem 'win32-event'
end

```


## Rails 3

In General - Rails 3 includes 'native' support for Bundler.
Newly-generated Rails apps come with a Gemfile that the user can edit, and Rails
automatically activates the bundle at boot.
If VCAP sees a `Gemfile.lock` in the application, it will ensure the needed gems are packaged, and set the `BUNDLE_PATH` environment variable to point at them.

Rails 3 Bundler support limitations – For simple lists of gem dependencies (anything that 'gem install' knows how to find), VCAP supports these well currently.
However, for complex lists, or build scripts for gems, being assembled at deploy time from several repos, VCAP has very limited support.
**As a rule, pre-package your app to the greatest possible extent.**



## Rails 2.3

Rails’ 'config.gem' mechanism, which allows the user to specify a list of gems that the app needs to operate,
is limited in that it cannot protect you from multi-deep dependencies (deeply layered gems using other gems).
In a cloud environment, this is an invitation for failure.

As mitigation, many Rails 2.3 applications have subsequently adopted Gem Bundler.

**VCAP currently does not 'detect' Rails 2.3 applications. If you need to run a
Ruby 2.3 app, disguise yourself as a Rails 3 app (by creating a config/application.rb file and a 'config.ru').**

## App Servers

The command-line and the deployed clouds currently only offer one app server for Sinatra and Rails applications: `Thin`.
However, if you are using Bundler, and nothing in your bundle requires Thin, VCAP cannot safely start your app using it.
In such cases, it will fall back to running your application using 'rails server', which uses WEBrick.

**For best performance and results, we suggest putting "gem 'thin'" in your Gemfile.**


## Ruby versions

VCAP supports two ruby 'runtimes':

- ruby18: Ruby version 1.8.7-p302 (or higher)
- ruby19: Ruby version 1.9.2-p180

**The default is ruby18**; you can specify otherwise using the `--runtime` flag at the vmc command-line:

```bash
$ vmc push myapp --runtime ruby19
```