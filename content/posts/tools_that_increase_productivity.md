---
title: "Tools that increase my productivity"
date: 2022-10-13T16:37:42-06:00
draft: true
author: Anthony Falzetti
categories:
- Programming
tags:
- software
- elixir
year: "2023"
month: "2022/10"
---

Apps:
1. Flycut
2. Skitch
3. Spectacle
4. Moom
5. Gitx
6. Insomnia
7. Postman
8. Iterm2
9. TablePlus

Apps I use everyday:

1. [Flycut](https://apps.apple.com/us/app/flycut-clipboard-manager/id442160987?mt=12) allows me to a copy buffer queue. This allows me to copy multiple things without thinking about it.
2. [Skitch](https://apps.apple.com/us/app/skitch-snap-mark-up-share/id425955336?mt=12) allows for marking up screenshots. This allows me to take a screenshot and draw arrows, or write text for explaining things or notating things for notes or messaging.
3. [Spectacle](https://www.spectacleapp.com/) is a application window manager. I have this setup so that I can manage my screen into quadrants.
4. [Moom](https://apps.apple.com/us/app/moom/id419330170?mt=12) similar to spectacle excepts I split my screen into 6 equal sizes for when I need even more application open at the same time.
5. [gitx](http://rowanj.github.io/gitx/) gui for git. This makes for simpler git commits when I have changes that I know I don't want to add in the same files as changes I do want to add.
6. [Insomnia](https://insomnia.rest/) allows for simple api requesting specifically for graphql, and since we have a compiled requests file
7. [Postman](https://www.postman.com/) Similar to Insomnia with some trade offs. Pros: can run requests from the command line, runner allows for running multiple queries in order. Cons: not as nice graphql workflow so have rarely used with our workflow. Use to use daily when working with a REST api.
8. [Iterm2](https://iterm2.com/) much stronger replacement for terminal.
9. [TablePlus](https://tableplus.com/) postgresql gui. I prefer seeing the results in a gui vs builtin pg terminal. Also stores previous queries, and allows for seeing table structures.
10. [VSCode](https://code.visualstudio.com/) Current editor of choice.

VSCode extensions:
1. WakaTime: I like to know how long I have been coding each day, and each week.
2. Vim: I prefer the vim keybindings
3. Project Manager: This allows for switching between projects without leaving the editor
4. Prettify JSON: Simple tool that allows for prettifying json (and elixir maps :) )
5. PlantUML: Sometimes when taking notes I create graphs or timeline. I like [plantuml](https://plantuml.com/) language as it tends to be dumb and doesn't have a huge learning curve.
6. Markdown All in One: I take all my notes in markdown, and this extension allows for:
  1. Tables to be formatted easily
  2. Code block syntax highlighting
7. Markdown Preview Enhanced: I only use this when my notes have PlantUML diagrams
8. File Utils: This allows for simple renaming files, duplicating files.
9. change-case: Simple extension that allows for changing from camel to snack_case without me retyping
10. code spell checker: Tells me when something I wrote isn't really a word: (recieved_at)
11. ElixirLS/erlang: Best extensions for working with elixir allow for go-to definition, and code completions.
12. ELM: Good extension that finds when the code won't compile (honestly I haven't spent enough time in elm to say its significance)
13. YAML/Better TOML: Since our configs are both in yaml and toml. Both languages care about whitespace and having an extension that shows when the indention is off is nice to have.

VScode Settings:
1. files.autoSave: "onFocusChange": Saves on focus lost. Totally a personal preference, but for me I tend to flip between editor, terminal, and UI. I forget to save changes especially in my notes, and I prefer to have everything I do to be persisted without me thinking about it.
2. Format On Save: Since our CI is strict on formatting I prefer it happen every time I save so I don't have to run `mix format` as part of my workflow. Also my notes have a lot of markdown tables and simply saving formats the tables appropriately.
3. Trim Auto Whitespace: I tend to use tab to do indentions and also at times I add an extra space, tab, or newline. This cleans up after me so I don't have to undo these as part of the git process.

