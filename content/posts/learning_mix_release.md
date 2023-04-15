---
title: "Learning mix release"
date: 2023-04-17T14:43:42-06:00
draft: true
author: Anthony Falzetti
categories:
- Programming
tags:
- elixir
- release
year: "2023"
month: "2023/04"
---

The problem I am attempting to solve is handling a misconfigured elixir application. Currently the workflow is make some changes that includes a feature flag or some other configuration that has a default. Run the release script which creates the package with the updated default configs. Then someone on the infra team deploys and checks to see if the application will run without crashing. If so then deploy complete even if the configurations didn't get merged correctly. My objective is to learn the config order and try to get the application to fail with warnings that point to a wiki.

<!--more-->

The deploy process I am attempting to recreate will look like this:
1. Create the package via `mix release`
2. `scp` the package to a docker container
3. run `sudo dpkg -i [package name]`
4. Smoke test and verify logs

Before I work on creating the release I want a simple docker container that will simulate the target environment. I am going to use alpine as a base since I don't plan to do much with this image and would like to save the space.

```dockerfile
# prepare release image
FROM alpine:3.17.3 AS app

# install runtime dependencies
RUN apk add --update bash openssl

RUN adduser -D --home /home/app --shell /bin/bash app

# # prepare app directory
RUN mkdir /opt/app
WORKDIR /opt/app

RUN chown -R app: /opt/app

USER app

CMD ["bash"]
```

Now that I have a target I want to create my package. Initially I am going to create the package locally then I will build it in a docker container so that I can eventually run it as a github action and have an artifact.

Since this is the first time I am setting up an elixir release for this project I must run `mix release.init` this will create a few files.

```shell
$ mix release.init
* creating rel/vm.args.eex
* creating rel/remote.vm.args.eex
* creating rel/env.sh.eex
* creating rel/env.bat.eex
```

Let's explore these files and what they do.

