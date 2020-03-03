# README

# Clone this repository

```
$ git clone github.com/wangsy/hello_docker_rails
```

# Create new rails

```
$ docker-compose run web bundle exec rails new . --force --no-deps --database=mysql --webpack=stimulus
```

# Build Again (with newly created rails source code)

```
$ docker-compose build
```

# DB Creation & Migration

```
$ cp database.yml config/
$ docker-compose run --rm web bundle exec rails db:create db:migrate
```

# Start running

```
$ docker-compose up
```
# Connect to server

http://localhost:3000