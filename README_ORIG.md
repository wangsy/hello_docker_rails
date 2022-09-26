# README

## Clone this repository

```
$ git clone git@github.com/wangsy/hello_docker_rails
```

or make following files

`Dockerfile`
```Dockerfile
FROM ruby:3.1.2
RUN apt-get update -qq && apt-get install -y curl

RUN curl -sL https://deb.nodesource.com/setup_18.x | bash -
RUN apt-get update -qq && apt-get install -y \
      nodejs \
      postgresql-client
RUN npm install --global yarn

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
version: '3'
services:
  redis:
    image: redis
  database:
    image: postgres
    volumes:
      - ./tmp/db:/var/lib/postgresql/data
    restart: always
    environment:
      - POSTGRES_PASSWORD=password
  web:
    build: .
    command: tail -f /dev/null
    volumes:
      - .:/myapp
      - gem_cache:/gems
    ports:
      - "3000:3000"
    environment:
      - POSTGRES_USERNAME=postgres
      - POSTGRES_PASSWORD=password
      - REDIS_URL=redis://redis:6379/0
    depends_on:
      - redis
      - database
volumes:
  db_data:
  gem_cache:
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
```Gemfile
source 'https://rubygems.org'
gem 'rails', '~>7'
```

and make `Gemfile.lock`
```sh
touch Gemfile.lock
```

## Create a new rails project

```sh
docker-compose run --no-deps web bundle exec rails new . --force -a propshaft -j esbuild --database=postgresql --skip-test --css tailwind
```

## Build again (with newly created rails source code, especially Gemfile)

```sh
docker-compose build
```

## DB Creation & Migration

```sh
cp database.yml config/
docker-compose run --rm web bundle exec rails db:create db:migrate
```

## Change Profile.dev
```
web: bin/rails server -p 3000 -b 0.0.0.0
js: yarn build --watch
css: yarn build:css --watch
```

## Start running

```sh
docker-compose up
docker compose exec web sh -c "bin/dev"
```
## Connect to server

http://localhost:3000

## Add TypeScript support
```
docker compose exec web yarn add --dev typescript tsc-watch
docker compose exec web yarn add --dev @typescript-eslint/parser
docker compose exec web yarn add --dev @typescript-eslint/eslint-plugin
```

## Sidekiq setting

`Gemfile`
```Gemfile
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
```sh
docker build -t hello_docker_rails .
```

create repository in ECR
```sh
aws ecr create-repository --repository-name hello_docker_rails --region ap-northeast-2
```

add repository tag to image
```sh
aws_account_id ==> registryId

docker tag hello-world aws_account_id.dkr.ecr.ap-northeast-2.amazonaws.com/hello_docker_rails
```

login to ECR
```sh
aws ecr get-login-password | docker login --username AWS --password-stdin aws_account_id.dkr.ecr.ap-northeast-2.amazonaws.com
```

push image to ECR
```sh
docker push aws_account_id.dkr.ecr.region.amazonaws.com/hello_docker_rails
```
