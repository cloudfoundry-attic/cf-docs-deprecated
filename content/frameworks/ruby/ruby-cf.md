---
title: Ruby, Rails and Cloud Foundry
description: Things to Know about Ruby, Rails and Cloud Foundry
tags:
    - ruby
    - rails
    - bundler
---

As we launch Cloud Foundry, a few elements of the framework and code are still being polished. This article provides some explanations and useful tips on things to watch and limitations.

Here are the highlights of the topics covered below:

+ Your application MUST supply a Gemfile that specifies ALL required gems
+ Some Gemfile features are not supported
+ VCAP does not currently support Isolate well
+ VCAP does not 'detect' Rails 2.3 applications (but there’s a workaround)
+ Stick to published gems
+ VCAP currently has no support for special Gemfile references
+ Pre-package your app to the greatest possible extent
+ Put "gem 'thin'" in your Gemfile
+ Default supported ruby version - ruby18 (Recommended1.8.7-p302 (or higher))
+ ruby19 is also supported – (Recommended ruby version 1.9.2-p174)


## Gems and Libraries

Some important comments about Gems and Gemfiles:

GEM list completeness - The proper way to interact with cloud foundry is to supply a complete Gemfile and then also run bundle package;bundle install each time you touch your Gemfile and prior to any vmc push or vmc update commands. **Your application MUST supply a Gemfile that specifies ALL required gems.**

The following Gemfile features are not supported -

+ gem dependencies on git urls or branches
+ gem :path => "some/path"
+ platform-conditional gems

Library dependency resolution - When a library you require has dependencies, you need to pay special attention into the way it tries to load dependent library versions (through $LOAD_PATH). **VCAP does not currently support Isolate well.**


## Rails 2.3

Rails’ 'config.gem' mechanism, which allows the user to specify a list of gems that the app needs to operate, is limited in that it cannot protect you from multi-deep dependencies (deeply layered gems using other gems). In a cloud environment, this is an invitation for failure.

As mitigation, many Rails 2.3 applications have subsequently adopted Gem Bundler.

**VCAP currently does not 'detect' Rails 2.3 applications. If you need to run a Ruby 2.3 app, disguise yourself as a Rails 3 app (by creating a config/application.rb file and a 'config.ru').**



## Bundler

In General – Watch out for complicated and confusing bundle settings. While VCAP tries to handle your configuration as best it can, it is possible to write Gemfiles that cannot be satisfied, such as requesting an old version of Rails and a brand-new version of Rack. These combinations fail immediately; you won't need to wait to push your app to the cloud to see it breaks. In the same token, Gemfiles and Gems that are too vague about versions can still create as well.

Bundler groups – Some forms of mixing multiple techniques for specifying which groups to load where (dev/test/prod) could cause unwanted results.

When in doubt, be more specific when calling 'Bundler.setup'. VCAP attempts to be as consistent as possible, **but it is still possible to confuse the Bundler with badly-behaved application bootstrap code.**

Gem local caching - The `bundle package` command will create a cache directory in your app and store local copies of the needed gems, allowing you to run `bundle install` with the --local flag for faster load time. This will not work in our case as the Bundler cannot currently package gems that need to be fetched from a git repository, and it does not attempt to package gems that you have manually specified a path to using the :path Gemfile option. **The best way to avoid this confusion is to stick to published gems.**

Special Gemfile references - Along with published gems, Gemfiles can refer to git repos by url, branch name, etc. And Bundler will build them using a specialized mechanism. **VCAP currently has no support for these at all, and apps that need them will fail.**


## Rails 3

In General - Rails 3 includes 'native' support for Bundler. Newly-generated Rails apps come with a Gemfile that the user can edit, and Rails automatically activates the bundle at boot. If VCAP sees a Gemfile.lock in the application, it will ensure the needed gems are packaged, and set the BUNDLE_PATH environment variable to point at them.

Rails 3 Bundler support limitations – For simple lists of gem dependencies (anything that 'gem install' knows how to find), VCAP supports these well currently. However, for complex lists, or build scripts for gems, being assembled at deploy time from several repos, VCAP has very limited support.  **As a rule, pre-package your app to the greatest possible extent.**



## App Servers

The command-line and the deployed clouds currently only offer one app server for Sinatra and Rails applications: Thin. However, if you are using Bundler, and nothing in your bundle requires Thin, VCAP cannot safely start your app using it. In such cases, it will fall back to running your application using 'rails server', which uses WEBrick. **For best performance and results, we suggest putting "gem 'thin'" in your Gemfile.**



## Ruby versions

VCAP supports two ruby 'runtimes':

ruby18: Recommended ruby version 1.8.7-p302 (or higher)
ruby19: Recommended ruby version 1.9.2-p174
**The default is ruby18**; you can specify otherwise using the --runtime flag at the vmc command-line:

```bash
$ vmc push myapp --runtime ruby19
```