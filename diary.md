# dev diary

## 2018-03-29

@dinosaur

- setup [ButtCloud.org](http://buttcloud.org) and various linked services (GitHub, Docker Cloud, Twitter, Gmail)
- started [`buttcloud/meta`](https://github.com/buttcloud/meta) repo to store any meta information (including [these dev diary entries](https://github.com/buttcloud/meta/blob/master/diary.md))
- started [`buttcloud/butt-landing](https://github.com/buttcloud/butt-landing) to be a simple public landing page for pub servers
  - working towards a multi-service pub using Docker Compose, i realized `ssb-viewer` and `git-ssb-web` were non-trivial to install in typical environments
  - the plan is to use this as a simple secondary service to get a multi-service pub working, then later can focus on the other secondary services

## 2018-03-30

@dinosaur

- made [Docker image](https://hub.docker.com/r/buttcloud/butt-landing) for `buttcloud/butt-landing`
- extracted minimal peer server and client code from `scuttlebot` into [`buttcloud/butt-peer`](https://github.com/buttcloud/butt-peer)
  - added custom logging plugin which uses [`pino`](https://github.com/pinojs/pino), to try and have a consistent logging system across services
  - made [two Docker images](https://hub.docker.com/r/buttcloud/butt-peer), one for `butt-peer-server` and one for `butt-peer-client`
    - i combined them so i could use the client code in the server healthcheck

## 2018-04-03

@dinosaur

- started `buttcloud/butt` as `docker-compose.yml` of peer server, landing server, and nginx proxy
- update `butt-peer-server` and `butt-peer-client` to not use `node` user, easier to start with default `root` user
  - why? i ran into an error with volume data permissions, would rather punt to later
  - UPDATE: changed this back, got it working with `node` user, needed to create volume in `Dockerfile` then mount external volume at same path
- update `butt-landing` to auto re-connect to sbot, so it doesn't error when sbot is not yet up
  - why? this is the recommended way to have docker services depend on each other, `docker-compose.yml` v3 not longer supports `depends_on: condition: healthy`
- setup Docker Hub automated builds to build tagged images based on git version tags
  - match tag name: `/^v[0-9.]+$/`, docker tag is same

## 2018-04-04

@dinosaur

- got minimal `buttcloud/butt` working!
- iron out some kinks...

## 2018-04-08

@dinosaur

- switch to focus on Docker Swarm
- change to use Traefik instead of Nginx Proxy

## 2018-04-09

@dinosaur

- keep trying to get the Swarm setup to work, grr...!

## 2018-04-13

@dinosaur

- take a break from Swarm for meow
- first pass at scaffolding [`buttcloud/buttcloud-provider`](https://github.com/buttcloud/buttcloud-provider)
  - have a working web server, browser app, and worker, but not yet complete to start feature development

## 2018-04-14

@dinosaur

- more progress toward provider app scaffolding

![buttcloud diary](./images/2018-04-14-buttcloud-landing.jpg)

## 2018-04-15 to 2018-04-17

@dinosaur

provider app has landed!

![provider app landing](./images/2018-04-17-buttcloud-landing.webm)

- integrate `redux-bundler`
- add emojis
- found seamless background image to tile on landing page
- start onboarding workflow
- implement start step of onboarding
  - validate forms on client
  - validate service calls on server, show errors in form
  - show success or failure messages as snackbar
  - after form submission (which creates the user)
    - generate json web token that identifies user
    - send welcome email to next page of onboarding with token
      - emails are sent by queuing a delayed job to a worker (`node-resque`)
      - setup decent email templates with `mjml`
    - store user in local storage in case they refresh page before progressing
    - show help text on page
  - allow user to resend onboarding email

## 2018-04-18

@ahdinosaur

back to the infra side, during breakfast this morning i figured out why `buttcloud/butt` was failing!

i had a hunch that it had to do with the address that `sbot` was binding to. i configured `host` as `example_butt-peer-server`, since that's how the Docker service was to be identified within the Docker network. but still, the health checker inside the service couldn't find it. i changed this to `0.0.0.0` and it works!

did the same for the landing service. now the stack comes online, you can `curl -H "Host: example.butt.nz" localhost` and get the output from the landing page associated with `example.butt.nz` (proxied by `traefik`).

next i added a [custom plugin to `butt-peer-server`](https://github.com/buttcloud/butt-peer/blob/3c4390907eebe18f98e5f5d9c839161b9d1e001e/server/plugins/address.js) that allows you to configure `externalHost`, in case it differs from `host`. this means we can bind to `host` (like `0.0.0.0`) but advertise our public multiserver address as `example.butt.nz` (like for invites).

then, on a whim from @mischa, i went [back to `buttcloud-provider` to swap `redux-form` for `final-form`](https://github.com/buttcloud/buttcloud-provider/pull/4), easy as.

made up some issues, want to step back to think about the next steps from here.
