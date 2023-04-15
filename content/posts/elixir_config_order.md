---
title: "Learning the Elixir Config ORder"
date: 2023-04-14T14:43:42-06:00
draft: false
author: Anthony Falzetti
categories:
- Programming
- Learning
tags:
- elixir
- config
year: "2023"
month: "2023/04"
---

The problem I am attempting to solve is handling a misconfigured elixir application. Currently, the workflow is to make some changes that include a feature flag or some other configuration that has a default. Run the release script which creates the package with the updated default configs. Then someone on the infra team deploys and checks to see if the application will run without crashing. If so then the deployment is successful even if the configurations didn't get merged correctly. My objective is to learn the config order.

<!--more-->

The deploy process I am attempting to recreate will look like this:
1. Create the package via `mix release`
2. `scp` the package to a docker container
3. run `sudo dpkg -i [package name]`
4. Smoke test and verify logs

### build-time configuration
Build-time configurations are evaluated at compile time which will be during the release.

Up until `mix release` was released if I wanted to have something configurable I would add it to config/config.exs. If it would be different for running locally versus production I would add it to the appropriate config/dev.exs or config/prod.exs.

This leads to the question of in what order are things evaluated. Take for instance I want to have a default config item that gets overwritten based upon the `MIX_ENV`.

```elixir
# config/config.exs
import Config

config :config_order,
  location: "config.exs"

import_config "#{config_env()}.exs"

# config/dev.exs
import Config

config :config_order,
  location: "dev.exs"
```

When I start the application and run `Application.get_env(:config_order, :location)` it returns "dev.exs" as I expect it to. What about if the config block is a partial keyword list:

```elixir
# config/config.exs
config :config_order, :partial_list,
  location: "config.exs",
  type: "config.exs"

# config/dev.exs
config :config_order, :partial_list, location: "dev.exs"
```

When I run `Application.get_env(:config_order, :config_items)` the application returns `[type: "config.exs", location: "dev.exs"]`. So it grabbed the `:type` from config.exs and the `:location` from dev.exs. Awesome! A pattern that I am seeing too often is shoving everything into all the config_envs as a copy and paste in an effort to ensure coverage, but in reality we should only be copying over the specific items that are different and leave the generic case in the config.exs.

So my takeaway is that the base case should go in the config.exs and anything that specifically should be different should be in the config_envs.

### Runtime configurations
Runtime configurations are evaluated at runtime and these are going to be specific to the instance. Two main ways of handling runtime configurations are putting them in the config/runtime.exs or using overlays.

```elixir
# config/config.exs
config :config_order, :partial_list,
  location: "config.exs",
  type: "config.exs"

# config/dev.exs
config :config_order, :partial_list, location: "dev.exs"

# config/runtime.exs
config :config_order, :partial_list, location: "runtime.exs"
```

This would be useful if we are evaluating something on the host machine such as hostname. Since this is still something that must be set before release there is still the limitation of not being able to make this change and restart the application once the app is deployed. During each deployment, this file will be overwritten this would not be an ideal place to put instance-level configurations.

Enter [config_providers](https://hexdocs.pm/elixir/main/Config.Provider.html). With a config_provider I can create a file on disk that contains configurations that will supersede all other configurations. This will be where we put instance-level configs that we want to be persisted between deployments.

If I make a new file /etc/app/overlays.exs that will be the configuration file that contains instance-level specific configurations.
```elixir
import Config

config :config_order,
  location: "overlays.exs"

config :config_order, :config_items,
  location: "overlays.exs",
  type: "overlays"
```

To get the overlays file to be evaluated simply add it as a config_provider in the mix.exs.

```elixir
releases: [
  prod: [
    config_providers: [
      {Config.Reader, path: "/etc/app/overlays.exs"}
    ],
    include_executables_for: [:unix],
    version: "0.1.0",
    steps: [:assemble, :tar]
  ]
]
```