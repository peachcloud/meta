# dev diary

## 2018-03-31

- setup [ButtCloud.org](http://buttcloud.org) and various linked services (GitHub, Docker Cloud, Twitter, Gmail)
- started [`buttcloud/meta`](https://github.com/buttcloud/meta) repo to store any meta information (including [these dev diary entries](https://github.com/buttcloud/meta/blob/master/diary.md))
- started [`buttcloud/butt-landing](https://github.com/buttcloud/butt-landing) to be a simple public landing page for pub servers
  - working towards a multi-service pub using Docker Compose, i realized `ssb-viewer` and `git-ssb-web` were non-trivial to install in typical environments
  - the plan is to use this as a simple secondary service to get a multi-service pub working, then later can focus on the other secondary services
