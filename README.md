# README

## Clone this repository

```
$ git clone git@github.com/wangsy/hello_docker_rails
```

## Create a new rails project

```
$ docker-compose run web bundle exec rails new . --skip --database=mysql --webpack=stimulus
```

## Build again (with newly created rails source code, especially Gemfile)

```
$ docker-compose build
```

## DB Creation & Migration

```
$ cp database.yml config/
$ docker-compose run --rm web bundle exec rails db:create db:migrate
```

## Start running

```
$ docker-compose up
```
## Connect to server

http://localhost:3000

## Sidekiq setting

`Gemfile`
```
gem 'sidekiq'
```

`config/application.rb`

```ruby
module Myapp
  class Application < Rails::Application
    ...
    config.active_job.queue_adapter = :sidekiq
```
