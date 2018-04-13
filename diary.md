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
