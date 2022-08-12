# README

## Clone this repository

```
$ git clone git@github.com/wangsy/hello_docker_rails
```

or make following files

`Dockerfile`
```Dockerfile
FROM ruby:3.1.2
RUN apt-get update -qq && apt-get install curl
RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
RUN echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list

RUN curl -sL https://deb.nodesource.com/setup_18.x | bash -
RUN apt-get update -qq && apt-get install -y \
      nodejs \
      yarn \
      postgresql-client

RUN mkdir /myapp
WORKDIR /myapp
COPY Gemfile /myapp/Gemfile
COPY Gemfile.lock /myapp/Gemfile.lock
ENV BUNDLE_PATH /gems
RUN gem update bundler && bundle install
COPY . /myapp

# Add a script to be executed every time the container starts.
COPY entrypoint.sh /usr/bin/entrypoint.sh
RUN chmod +x /usr/bin/entrypoint.sh
ENTRYPOINT ["entrypoint.sh"]

# Start the main process.
CMD ["bundle", "exec", "rails", "server", "-b", "0.0.0.0"]
```

`docker-compose.yml`
```
version: "3.9"
services:
  db:
    image: postgres
    volumes:
      - ./tmp/db:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: password
  web:
    build: .
    command: bash -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 3000 -b '0.0.0.0'"
    volumes:
      - .:/myapp
    ports:
      - "3000:3000"
    depends_on:
      - db
```

`entrypoint.sh`
```sh
#!/bin/bash
set -e

# Remove a potentially pre-existing server.pid for Rails.
rm -f /myapp/tmp/pids/server.pid

# Then exec the container's main process (what's set as CMD in the Dockerfile).
exec "$@"
```

`Gemfile`
```
source 'https://rubygems.org'
gem 'rails', '~>7'
```

and make `Gemfile.lock`
```sh
$ touch Gemfile.lock
```

and then
```sh
$ docker compose run --no-deps web rails new . --force --database=postgresql

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

## Push image to ECR

add tag to docker image
```
docker build -t hello_docker_rails .
```

create repository in ECR
```
aws ecr create-repository --repository-name hello_docker_rails --region ap-northeast-2
```

add repository tag to image
```
aws_account_id ==> registryId

docker tag hello-world aws_account_id.dkr.ecr.ap-northeast-2.amazonaws.com/hello_docker_rails
```

login to ECR
```
aws ecr get-login-password | docker login --username AWS --password-stdin aws_account_id.dkr.ecr.ap-northeast-2.amazonaws.com
```

push image to ECR
```
docker push aws_account_id.dkr.ecr.region.amazonaws.com/hello_docker_rails
```