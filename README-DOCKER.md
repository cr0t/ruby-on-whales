# Ruby on Whales

It's our humble attempt to use and adjust for our needs instructions we found in a great Evil Martians article [Dockerizing Ruby and Rails development]. This repository should be used as template for new Ruby or Ruby on Rails projects (we need to install additional gems for the latter). Also, it is possible to comment out (or delete) unnecessary services that you do not need to have in simple Ruby projects (for example, `sidekiq`, or `redis`, or everything except the app container).

## Tips&Tricks

`docker-compose up`

The main command we will be using is , which attempts to init all the services described in the configuration file. If it's just the first time user runs the command, it will download and install all the prerequisites, build the app image, and set environment up and running.

`docker-compose exec shell bash`

Runs a bash session in the app environment (so you can use `ruby`, `irb`, `bundle`, `rails`, `rake`, and many other commands.

## New Ruby on Rails Application

Here is the way of we can use this approach to create a fresh new Ruby on Rails application. We consider that you just cloned this repository and have nothing else in the current directory.

```bash
$ git clone git@github.com:cr0t/ruby-on-whales.git <your-app-name>

$ cd <your-app-name> && rm -rf .git

# now you can open docker-compose.yml, remove services you do not need, update Postgresql password update Ruby/Node/etc. versions

$ docker-compose up
```

This step takes some time to download and build images. If it went well you should see a lot of output and eventually notifications about running services (depends on which you left in the configuration file).

**In a separate terminal** you need to start a new shell session to install Ruby on Rails gem and other libraries you want.

```bash
$ docker-compose exec shell bash

# this gives us a bash session inside our Docker containers, so we can run our favourite ruby/rails commands, for example:

root@11f8d8b944bb:/app# gem install rails
root@11f8d8b944bb:/app# rails new . --database=postgresql
```

...are we there yet?!

Almost. Let's install Sidekiq and configure our new lovely Ruby on Rails app to use the shiny Docker environment.

We need to install `sidekiq` and configure it, also, we need to update `database.yml`, so our Ruby on Rails app can use it:

```bash
root@11f8d8b944bb:/app# vim Gemfile # yes, we can edit Gemfile inside container, but you can do it on the host machine too
root@11f8d8b944bb:/app# bundle install
root@11f8d8b944bb:/app# vim config/initializers/sidekiq.rb
root@11f8d8b944bb:/app# cat config/initializers/sidekiq.rb
Sidekiq.configure_server do |config|
  config.redis = { url: 'redis://redis:6379/0'  }
end

Sidekiq.configure_client do |config|
  config.redis = { url: 'redis://redis:6379/0'  }
end
root@11f8d8b944bb:/app# vim config/sidekiq.yml
root@11f8d8b944bb:/app# cat config/sidekiq.yml
---
:concurrency: 5
staging:
  :concurrency: 10
production:
  :concurrency: 20
:queues:
  - critical
  - default
  - low
root@11f8d8b944bb:/app# vim config/database.yml
root@11f8d8b944bb:/app# cat config/database.yml
# ...
default: &default
  adapter: postgresql
  encoding: unicode
  # For details on connection pooling, see Rails configuration guide
  # https://guides.rubyonrails.org/configuring.html#database-pooling
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  username: postgres
  password: sec-ruby-on-whales-ret
  host: postgres
# ...
root@11f8d8b944bb:/app# rake db:create
root@11f8d8b944bb:/app# exit
$ docker-compose stop # or down
$ docker-compose start # or up
```

> We need to run two last commands to re-start all services defined in `docker-compose.yml` (if we left it original).

Ok! Now you can try to open http://localhost:3000/ in the browser on the host machine and check if you see Ruby on Rails welcome page.

[Dockerizing Ruby and Rails development]: https://evilmartians.com/chronicles/ruby-on-whales-docker-for-ruby-rails-development
