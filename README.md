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