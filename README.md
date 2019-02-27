# API Gateway with Kong

This repo contains script to deploy Kong API Gateway on you server

## Install

1. Clone `.env.example` into `.env`
2. Edit your approriate values in `.env`
3. Run `docker-compose up -d`

## First login to Konga - a Kong GUI

1. Access `you-host.com:1337`, this is default port of Konga
2. Create your admin account
3. You'll redirected to login page, use your admin account to login
4. Add a new connection, give the name and admin url `http://kong:8001/`
5. Start config Kong on KongA. At the second login, you don't need to do 1,2,3,4 above.