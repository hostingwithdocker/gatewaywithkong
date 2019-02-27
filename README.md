# API Gateway with Kong

This repo contains script to deploy Kong API Gateway on you server

## Install

1. Clone `.env.example` into `.env`
2. Edit your approriate values in `.env`
3. Remember to set the variable `KONGA_ENV=development` to init Konga database
4. Run `docker-compose up -d`
5. Check `docker-compose ps`.
If there is less than 3 containers up, then run `docker-compose down` and up again. 
If 3 container are up. Continue the next step.
6. Wait a moment then down the stack: `docker-compose down` then you done the installation.

> If you meet any problems, let remove the directory `/postgres` the repeat the step 1

## First login to Konga - a Kong GUI

1. Remember to set the variable `KONGA_ENV=production` (no init Konga database)
2. Run `docker-compose up -d`
3. Access `you-host.com:1337`, this is default port of Konga
4. Create your admin account
5. You'll redirected to login page, use your admin account to login
6. Add a new connection, give the name and admin url `http://kong:8001/`
7. Start config Kong on KongA. At the second login, you don't need to do 1,2,3,4 above.