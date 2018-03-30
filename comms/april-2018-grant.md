# ButtCloud  :partly_sunny: :rainbow:

_an April 2018 #ssbc-grants proposal_

i'd like to work on better pub infrastructure and a hosted pub-as-a-service product.

## team

me! :smiley_cat:

## goals

- we build an [open source business](http://blog.dinosaur.is/workers-of-open-source-unite/) on Scuttlebutt!
- a pub is [your personal cloud device](%Gwqklkj0b2CBT5tPiz5170NWsPp3xiuLbOImEaG/e+4=.sha256) that is always available and publicly accessible
- a pub should be affordable (start pricing at ~$7/month per pub, try to get to ~$1/month per pub)
- we support open source infrastructure providers, like [OVH](https://www.ovh.com/world/public-cloud/) and [Catalyst](http://catalyst.net.nz/catalyst-cloud)
- the infrastructure code should easy for others to contribute and maintain 

## the story so far

pubs are important ([1](%f6ZRXO2t0rwUw/lq5FpWCHtuHc9406Q37TB+lF9bUbc=.sha256), [2](https://writtenby.adriengiboire.com/2018/03/14/my-first-week-experience-with-scuttlebutt-and-patchwork/), [3](https://twitter.com/nicolasini/status/974364249219727360)), but [public pubs are a stop-gap we've used for too long](%MZCHPVkh8sNhTsevWgSVXNlL6dYgSnzBvB3hJcJZ/7k=.sha256), we need [private pubs for everyone](%Gwqklkj0b2CBT5tPiz5170NWsPp3xiuLbOImEaG/e+4=.sha256)!

[over the holidays](%1TVZigDql9VAQaZZVX/QegKan18urBuXQikOWE1uTMk=.sha256), i got maybe 80% of the way towards automated Scuttlebutt pub infrastructure using [Salt Stack](https://docs.saltstack.com/), hosted on [Open Stack](https://docs.openstack.org) using [OVH public cloud](https://www.ovh.com/world/public-cloud/). i've also been able to achieve _mostly_ reliable pubs using an [`ssb-pub`](https://github.com/ahdinosaur/ssb-pub) Docker image.

i want to throw away all the [Salt Stack](https://docs.saltstack.com/) work i did and start over with [Docker Swarm](https://docs.docker.com/engine/swarm/).

i expect this will take [longer than a month of full-time work to complete](%9ZzHJ2F0MHncqLLInC47Tp/OuHEUcHyRfWYierUpUKc=.sha256).

## sub-projects

there are three main sub-projects:

- provider service
- hub swarm
- pub service

## architecture

- provider service
  - web app
    - join
      - land
      - sign in
      - create pub
      - pay for pub
      - start pub service
    - monitor
      - land
      - sign in
      - view pub
      - see stats
    - command
      - land
      - sign in
      - view pub
      - run command
  - web api
    - users
      - id
      - name
      - email
    - pub
      - bots
        - id
        - userId
        - name
        - status (up, down, none)
      - stats 
        - stream Docker stats
      - commands
        - relay commands to pub services
      - orchestrator
        - on schedule, check what pubs are up
        - have 1 pub per 1 GB memory, 1 hub per 15 GB memory
        - queue worker jobs to ensure correct swarm
    - payment
      - products
      - plans
      - customers
      - subscriptions
  - swarm worker
    - manage hub [machines](https://docs.docker.com/machine/drivers/openstack/)
      - create hub
      - destroy hub
    - manage pub [services](https://docs.docker.com/engine/swarm/swarm-tutorial/deploy-service/)
      - ensure pub service is up
      - ensure pub service is down
  - mailer worker
- pub service

## stack

- provider service
  - web api
    - [@feathersjs/socketio](https://github.com/feathersjs/socketio)
    - [@feathersjs/authentication](https://github.com/feathersjs/authentication)
    - [@feathersjs/authentication-jwt](https://github.com/feathersjs/authentication-jwt)
    - [feathers-stripe](https://github.com/feathersjs-ecosystem/feathers-stripe)
    - [node-resque](https://github.com/taskrabbit/node-resque)
    - [docker-remote-api](https://github.com/mafintosh/docker-remote-api)
  - web app
    - [next.js](https://github.com/zeit/next.js/)
    - [ramda](http://ramdajs.com/docs/)
    - [@feathersjs/socketio-client](https://github.com/feathersjs/socketio-client)
    - [@feathersjs/authentication-client](https://github.com/feathersjs/authentication-client)
    - [react](https://facebook.github.io/react)
    - [react-hyperscript](https://github.com/mlmorg/react-hyperscript)
    - [recompose](https://github.com/acdlite/recompose)
    - [fela](https://github.com/rofrischmann/fela)
    - [material-ui](https://material-ui.com/)
    - [react-stripe-elements](https://github.com/stripe/react-stripe-elements)
  - swarm worker
    - [node-resque](https://github.com/taskrabbit/node-resque)
    - [docker-remote-api](https://github.com/mafintosh/docker-remote-api)
  - mailer worker
    - [node-resque](https://github.com/taskrabbit/node-resque)
    - [nodemailer](https://github.com/nodemailer/nodemailer)
    - third-party: [sendgrid](https://sendgrid.com/)
    - dev tool: [maildev](https://github.com/djfarrelly/maildev)
- [pub service](http://github.com/ahdinosaur/ssb-pub)
  - [scuttlebot](https://github.com/ssbc/scuttlebot)
  - [ssb-viewer](%MeCTQrz9uszf9EZoTnKCeFeIedhnKWuB3JHW2l1g9NA=.sha256)
  - [git-ssb-web](%q5d5Du+9WkaSdjc8aJPZm+jMrqgo0tmfR+RcX5ZZ6H4=.sha256)

## roadmap

_rough draft, subject to change_

- upgrade pub service
  - update `ssb-pub` to use docker-compose (so we can host multiple Scuttlebutt processes in the same service)
    - add `ssb-viewer`
    - add `git-ssb-web`
- prototype hub swarm
  - setup single "hub manager" to be docker swarm manager
    - run mock provider service on manager
  - setup many "hub worker"s to be docker swarm workers
- create provider service
  - upgrade mock provider service to include postgres and redis databases
  - scaffold web api, web app, swarm worker, and mailer worker
  - implement provider web app user flows
- automate swarm
  - implement swarm orchestration functionality
- start business
  - understand costs, determine prices, forecast profit/loss
  - decide on company jurisdiction and legal structure
  - [copy](https://getterms.io/) or create (with help) a Terms of Service & Privacy Policy
