---
title: Using MongoDB and GridFS with Ruby
description: Ruby Development with MongoDB and GridFS
tags:
    - mongodb
    - grid-fs
    - rails
    - tutorial
---

GridFS is a specification for storing files larger than the MongoDB size limit in MongoDB databases. It works by storing the file in chunks, which are managed through an API that treats the collection of chunks as a single ojbect. This tutorial shows how to use GridFS in a Rails application by adding an avatar to the User model.

Many web applications allow users to upload and retrieve generated images, such as profile pictures, photos, or video thumbnails. If you are using Ruby on Rails there are a couple of great frameworks you can use: PaperClip and CarrierWave.

CarrierWave was selected for this project because it seems the least obtrusive and most flexible. CarrierWave can be used with the file system, Amazon S3, or a database, including Mongo GridFS.

The first task is to add images when users register on the app using their Facebook account. Here are the steps to support uploading and serving images. For more details on the Facebook integration review the `user.rb` model in the source code.

## Steps to Perform on Your Terminal

This assumes you already have a Ruby on Rails 3.0 application on Cloud Foundry with a Users model.

```bash

# Log in to Cloud Foundry if you are not logged in
$ vmc login youremail@domain.com

$ vmc create-service mongodb

# See what the newly created mongo service is called
$ vmc services

# Bind the service to your existing application
$ vmc bind-service mongodb-???? appname

```

## Steps on Your Code Base

### Dependencies

Add the following gems to your Gemfile

``` ruby
gem 'carrierwave'
gem 'carrierwave-mongoid', :require => "carrierwave/mongoid"
```

Generate the uploader

```bash
$ rails generate uploader Avatar
```

Edit the generated file, `app/uploaders/avadar_uploader.rb`, to use grid_fs

``` ruby
class AvatarUploader < CarrierWave::Uploader::Base

    # Choose what kind of storage to use for this uploader:
    storage :grid_fs

    # Override the directory where uploaded files will be stored.
    # This is a sensible default for uploaders that are meant to be mounted:
    def store_dir
        "#{model.class.to_s.underscore}/#{mounted_as}/#{model.id}"
    end

    # Provide a default URL as a default if there hasn't been a file uploaded:
    def default_url
        "/images/fallback/" + [version_name, "default.png"].compact.join('_')
    end
end
```

Update your ActiveRecord model to store the avatar.

Make sure you are loading CarrierWave after loading the ORM, otherwise you need to require the relevant extension manually, for example:

``` ruby
require 'carrierwave/orm/activerecord'
```

Add a string column to the model on which you want to mount the uploader.

```bash
$ add_column :users, :avatar, :string
```

Open your model file and add code to mount the uploader:

``` ruby
class User
...
    mount_uploader :avatar, AvatarUploader

    # Make sure that the avatar is accessible
    attr_accessible :emails, :avatar, :remote_avatar_url, :email, :last_name #....
...
end
```

Create an initializer for Mongoid to use your MongoDB instance on Cloud Foundry. Name it `01_mongoid.rb` so it runs before everything else.

``` ruby
Mongoid.configure do |config|
  conn_info = nil

  if ENV['VCAP_SERVICES']
    services = JSON.parse(ENV['VCAP_SERVICES'])
    services.each do |service_version, bindings|
      bindings.each do |binding|
        if binding['label'] =~ /mongo/i
          conn_info = binding['credentials']
          break
        end
      end
    end
    raise "could not find connection info for mongo" unless conn_info
  else
    conn_info = {'hostname' => 'localhost', 'port' => 27017}
  end

  cnx = Mongo::Connection.new(conn_info['hostname'], conn_info['port'], :pool_size => 5, :timeout => 5)
  db = cnx['db']
  if conn_info['username'] and conn_info['password']
    db.authenticate(conn_info['username'], conn_info['password'])
  end

  config.master = db
end
```

Update your CarrierWave initializer to use the Cloud Foundry MongoDB service.

``` ruby
#initializers/carrierwave.rb
require 'serve_gridfs_image'

CarrierWave.configure do |config|
  config.storage = :grid_fs
  config.grid_fs_connection = Mongoid.database

  # Storage access url
  config.grid_fs_access_url = "/grid"
end
```

Handle requests for the images in `lib/serve_gridfs_image.rb`:

``` ruby
class ServeGridfsImage
  def initialize(app)
      @app = app
  end

  def call(env)
    if env["PATH_INFO"] =~ /^\/grid\/(.+)$/
      process_request(env, $1)
    else
      @app.call(env)
    end
  end

  private
  def process_request(env, key)
    begin
      Mongo::GridFileSystem.new(Mongoid.database).open(key, 'r') do |file|
        [200, { 'Content-Type' => file.content_type }, [file.read]]
      end
    rescue
      [404, { 'Content-Type' => 'text/plain' }, ['File not found.']]
    end
  end
end
```

## Deploy

```bash
$ bundle install
    bundle package
    vmc update app_name
```

## Conclusion

This will give you the ability to upload and serve images. Do note that it does not provide image resizing. If you are using devise for example you can import the avatar(profile picture) of the user when they sign up.

``` ruby
class << self
    def new_with_session(params, session)
      super.tap do |user|
        if session['devise.omniauth_info']
          if data = session['devise.omniauth_info']['user_info']
            user.display_name = data['name'] if data.has_key? 'name'
            user.email = data['email']
            user.username = data['nickname'] if data.has_key? 'nickname'
            user.first_name = data['first_name'] if data.has_key? 'first_name'
            user.last_name = data['last_name'] if data.has_key? 'last_name'
            user.remote_avatar_url = data['image'] if data.has_key? 'image'
          end
        end
      end
    end
  end
```

## References

+ [Connecting to a MongoDB Database](http://support.cloudfoundry.com/entries/20016922-connecting-to-a-mongo-db)
+ [CarrierWave](https://github.com/jnicklas/carrierwave)
+ [CarrierWave for Mongoid](https://github.com/jnicklas/carrierwave-mongoid)
+ [Setting up CarrierWave file uploads using GridFS on Rails 3 and mongoid](http://antekpiechnik.com/posts/setting-up-carrierwave-file-uploads-using-gridfs-on-rails-3-and-mongoid)
+ [Devise](https://github.com/plataformatec/devise)