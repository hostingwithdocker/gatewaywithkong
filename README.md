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