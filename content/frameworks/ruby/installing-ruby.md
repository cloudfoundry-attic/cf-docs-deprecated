---
title: Installing Ruby and RubyGems
description: How to Install Ruby and RubyGems
tags:
    - tutorial
    - ruby
    - vmc
    - Gemfile
---

The following sections provide basic information about installing Ruby and RubyGems on Windows and a variety of Linux computers.

## Windows
Download and install [Ruby Installer for Windows](http://www.rubyinstaller.org/ "ruby installer for windows"). The installer already includes RubyGems.

Be sure you use the Ruby-enabled command prompt window when you later install and use `vmc`.   You access this command prompt from the Windows Start menu (**All Programs > Ruby \<version\> > Start Command Prompt with Ruby**).

Finally, update RubyGems from the Ruby Command Prompt:

    prompt> gem update --system

### Support for Windows Gemfiles

Once you have Ruby installed, you can follow the instructions on [Ruby Apps on Cloud Foundry](ruby-cf.html) to learn the details of pushing an app and working with your Gemfile

One thing to note for Windows users is the following:

When a `Gemfile.lock` is generated on a Windows machine, it often contains gems with Windows-specific versions.
Versions of gems such as mysql2, thin, and pg end up containing “-x86-mingw32”.

For example, running `bundle install` on a Windows machine with a Gemfile that looks like this:

``` ruby
gem ‘sinatra’
gem ‘mysql2’
gem ‘json’
```

Would result in a Gemfile.lock that looks like this:

``` ruby
GEM
  remote: http://rubygems.org/
  specs:
    json (1.7.3)
    mysql2 (0.3.11-x86-mingw32)
    rack (1.4.1)
    rack-protection (1.2.0)
      rack
    sinatra (1.3.2)
      rack (~> 1.3, >= 1.3.6)
      rack-protection (~> 1.2)
      tilt (~> 1.3, >= 1.3.3)
    tilt (1.3.3)

PLATFORMS
  x86-mingw32

DEPENDENCIES
  json
  mysql2
  sinatra

```

Now Cloud Foundry will be able to properly install these gems without the need to modify your `Gemfile.lock`

## Mac OS X
Version 10.5 and higher of Mac OS X already ships with Ruby and RubyGems installed.

If you are using an earlier version of Mac OS, download and install the latest version of [Ruby](http://www.ruby-lang.org/en/downloads/ "ruby source code") and then [RubyGems](http://rubygems.org/pages/download).

## Ubuntu

From a terminal, use the `apt-get` command-line tool to install Ruby and RubyGems, as shown below.

1. Install the full Ruby package and RubyGems:

    `prompt$ sudo apt-get install ruby-full rubygems`

    Consult your system administrator for any required authentication credentials for the `sudo` command.

2.  Test to ensure that the `gem` command is in your path:

    `prompt$ which gem`

    If the command is not found, then update your `PATH` variable accordingly.  For example, you can update your `.bashrc` file with the following line:

    `export PATH=$PATH:/var/lib/gems/1.8/bin`

3.	Update RubyGems:

    Ubuntu 10.04

        prompt$ sudo gem install rubygems-update
        prompt$ sudo /var/lib/gems/1.8/bin/update_rubygems

    Ubuntu 11.10

        prompt$ sudo su -
        prompt# export REALLY_GEM_UPDATE_SYSTEM=true
        prompt# gem update --system
        prompt# exit

## RedHat/Fedora

From a terminal, use the `yum` command-line tool to install Ruby and RubyGems, as shown below.

1. Install Ruby:

    `prompt$ sudo yum install ruby`

2.  If you are using RedHat Enterprise Linux 6, enable the *Optional* channel for your host by logging into [Red Hat Network (RHN)](https://rhn.redhat.com/).

3. Install RubyGems:

    `prompt$ sudo yum install rubygems`

## Centos
From a terminal, use the `yum` command-line tool to install Ruby and RubyGems, as shown below.

1. Install the basic Ruby package:

    `prompt$ yum install -y ruby`

2. Install additional Ruby packages and documentation:

    `prompt$ yum install -y ruby-devel ruby-docs ruby-ri ruby-rdoc`

3. Install RubyGems:

    `prompt$ yum install -y rubygems`

## SuSE

From a terminal, use the `yast` command-line tool to install Ruby and RubyGems, as shown below.

1. Install Ruby:

    `prompt$ yast -i ruby`

2. Install RubyGems:

    `prompt$ yast -i rubygems`

## Debian

You use Ruby Version Manager (`rvm`) to install Ruby and RubyGems on Debian.  The following procedure shows how to install `rvm` if you have not already done so.

1.  Use the following `apt-get` command-line tool to install the required packages:

    `prompt$ sudo apt-get install gcccurl git-core build-essential libssl-dev libreadline5 libreadline5-dev zlib1g zlib1g-dev`

2.  Run the `bash` script to install `rvm` from [Ruby Version Manager](https://rvm.beginrescueend.com/install/rvm).

    `prompt$ bash << curl -s https://rvm.beginrescueend.com/install/rvm`

3.  Edit your *~/.bashrc* file as described by the RVM installation in the precding step.

4. Use `rvm` to install Ruby and RubyGems as shown:

    `prompt$ rvm package install zlib`

    `prompt$ rvm install 1.9.2 -C --with-zlib-dir=$rvm_path/usr`

    `prompt$ rvm use 1.9.2`
