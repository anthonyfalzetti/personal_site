---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
draft: true
author: Anthony Falzetti
categories:
- Personal
- 75 Hard
- Programming
tags:
- software
- elixir
year: "{{ dateFormat "2006" .Date }}"
month: "{{ dateFormat "2006/01" .Date }}"

---
Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor
incididunt ut labore et dolore magna aliqua.

<!--more-->

Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut
aliquip ex ea commodo consequat.
