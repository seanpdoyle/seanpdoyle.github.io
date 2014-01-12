---
layout: post
title: Remote Rails Commands with Recap-Console
modified: 2014-01-12
---

Run `rails (db)console` commands with [`Recap`](https://github.com/freerange/recap).

[`Recap`](https://github.com/freerange/recap) is a reinterpretation of the [`capistrano`](https://github.com/capistrano/capistrano/wiki) deployment and multi-server management gem.

## The Setup

In your `Capfile`, require the library.

```ruby
# Capfile
require 'recap/tasks/rails/console'
```

## The Payoff

The remote consoles will run on the first available server with the `app` role.

#### Rails commands

To run the remote equivalent of `rails console`, run the following.

```console
$ cap rails:console
```

To run the remote equivalent of `rails dbconsole`, run the following.

```console
$ cap rails:dbconsole
```

#### SSH

In addition to `rails` commands, you can open a vanilla `ssh` console.

```console
$ cap ssh:console
```

Finally, to run an arbitrary shell command, pass the command in via the `RUN` variable. Note that the current directory is your recap project directory.

```console
$ cap ssh:command RUN="tail -f log/production.log"
```

#### Additional Reading

* `Capistrano` [version 3](http://capistranorb.com) now uses `git` for server-side deployment file resolution.
