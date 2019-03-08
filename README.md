# API Gateway with Kong

This repo contains script to deploy Kong API Gateway on you server.

## Install

1. Clone `.env.example` into `.env`
2. Edit your appropriate values in `.env`
3. Remember to set the variable `KONGA_ENV=development` to generate Konga's database
4. Run `docker-compose up -d`
5. Check `docker-compose ps`
If there are less than 3 containers up, you have to run `docker-compose down` and up again
Until 3 containers are up. Continue the next step
6. To complete the installation process, wait a moment then down the stack: `docker-compose down`

> If you meet any problems continuously, let remove the directory `/postgres` before start again at step 1

## First login to Konga - a Kong GUI

1. Remember to set the variable `KONGA_ENV=production` (no need to init Konga database)
2. Run `docker-compose up -d`
3. Access `you-host.com:1337`, this is default port of Konga
4. Create your admin account
5. You'll be redirected to login page, use your admin account to login
6. Add a new connection, give the name and admin url `http://kong:8001/`
7. Start config Kong on KongA
8. At the second login, you don't need to do steps above

## Explain the stack

### Components and words:

#### 1. The API Gateway 
The API Gateway takes all the requests from the clients, then routes them to the appropriate microservice with request routing, composition, and protocol translation. Typically it handles a request by invoking multiple microservices and aggregating the results, to determine the best path. It can translate between web protocols and web-unfriendly protocols that are used internally. ([more](https://www.nginx.com/learn/api-gateway/))

#### 2. Kong 
Kong is deployed on top of reliable technologies like NGINX and Apache Cassandra or PostgreSQL, and provides you with an easy-to-use RESTful API to operate and configure the system. Once Kong is running, every client request being made to the API will hit Kong first and then be proxied to the final API. In between requests and responses Kong will execute any installed plugins, extending the API feature set. Kong effectively becomes the entry point for every API request. ([more](https://konghq.com/solutions/gateway/))

#### 3. Konga
An Elegant Kong GUI, open source tool that enables you to manage your Kong API Gateway with ease ([more](https://pantsel.github.io/konga/)). The original Kong Api Gateway works with an Admin RESTful API. It has no native UI to interact with, but there is an Enterprise UI tool you can paid alternatively Konga.

### Take a look on the file docker-compose.yml
There are 4 parts as known as services inside:

#### 1. kong-database
In this part, we use Postgres for database. In the environment declarations, we need to set the default user and password, which we also use for the setting of Kong and Konga. Postgres has an utility for health-check, which is called `pg_isready`. Three most significant configs:
  
  - POSTGRES_USER: Username for Kong service using
  - POSTGRES_PASSWORD: Password or postgres user
  - POSTGRES_DB: Database name of Kong where Kong storages it's configuration

This service will map the directory `./postgre` in current working dir for persistent volume data. You will need Kong to keep data permanent except you wish install Kong again and again every time start it up.

#### 2. kong-prepare
If we installed Kong manually, we could need to run command `kong migrations bootstrap` to init the database structure of Kong before starting it. Now we are deploying it automatically, so we have to build a helper container which will run that command for us but It must delete itself after finishing the task. Do you remember about container's life time? A container is actually a Linux namespace which isolate its own process from host. When the process goes to the end of life, the container is also terminated. The configuration is the same the next session, look bellow or see it `docker-compose.yml`.

#### 3. kong
Kong can start itself but after Kong's Database ready. To force it wait for DB ready, we write down what it depends on, what it has to wait for.
  ```yaml
    depends_on:
      - kong-database
      - kong-prepare
  ```
To custom Kong installation, we should change the value of environment variables. There are some important config that you need to notice:
  - KONG_DATABASE: 2 choices here "cassandra" or "postgres"
  - KONG_CASSANDRA_CONTACT_POINTS: it's the name of db service about "kong-database"
  - KONG_PG_HOST: same as KONG_CASSANDRA_CONTACT_POINTS
  - KONG_PG_DATABASE: the db name for Kong
  - KONG_PG_USER: Postgres's user we made above
  - KONG_PG_PASSWORD: Password of db user
  - KONG_PROXY_ACCESS_LOG: This is the default place to write down log. You should fill it `/dev/stdout` for monitoring it from the host.
  - KONG_ADMIN_ACCESS_LOG: same above
  - KONG_PROXY_ERROR_LOG: You should fill it `/dev/stderr` 
  - KONG_ADMIN_ERROR_LOG: save above
  - KONG_ADMIN_LISTEN: 0.0.0.0:8001 , this is the default binding of Admin API on HTTP
  - KONG_ADMIN_LISTEN_SSL: 0.0.0.0:8444, this is the default binding of Admin API on HTTPS
  - KONG_PROXY_LISTEN: 0.0.0.0:8000, this is the default binding of Kong Proxy on HTTP
  - KONG_PROXY_LISTEN_SSL: 0.0.0.0:8443, this is the default binding of Kong Proxy on HTTPS

As config above, we see Kong exposing 4 ports but we only need to publish 2 proxy port to the Internet. Let keep admin port securely in local network.

#### 4. konga-prepare
Konga need a ready db structure to work with. Just same as `kong-prepare` above, we build a helper container to run the command `node index.js -c prepare` for us. See full command [here](https://github.com/pantsel/konga#running-konga).

#### 5. konga
Lastly, we have to install `konga` for management of Kong's configuration. This service depends on `kong-database` and `kong`. You have to list them inside docker-compose.yml. There are needed configs:
  - NODE_ENV: "production" or "development"
  - TOKEN_SECRET: random string
  - DB_ADAPTER: "postgres" or "mysql"
  - DB_HOST: "kong-database" is the name of database service above. Just use service name, no IP or container name.
  - DB_PORT: the port of db service
  - DB_USER: Postgres user above
  - DB_PASSWORD: Postgres user's password
  - DB_DATABASE: `konga-db` is the name of Konga database, don't confuse with kong's database.
  - DB_PG_SCHEMA: `konga-schema` only use in Postgres
  - SSL_KEY_PATH: `/cert/server.crt` The path to private key if you wish to run Konga on HTTPS
  - SSL_CRT_PATH: `/cert/server.crt` The path to public key if you wish to run Konga on HTTPS

Keep in mind, ssl key path here is the internal path inside the container, so you have to map the host dir where contains keys to the container dir as `/cert`:
  ```yaml
    volumes:
      - ./cert:/cert
  ```

You're free to edit this setting  by your self. Make it online by commands:

  ```bash
  # start the stack
  docker-compose up -d

  # stop the stack
  docker-compose stop

  # Or stop and remove all containers
  docker-compose down
  ```