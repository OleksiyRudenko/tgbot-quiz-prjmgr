# Project Management Skiilset Quiz Telegram Bot

The bot runs a quiz, stores results, 
suggests a project management skill set assessment.

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
## Table of Contents

- [Requirements Addressed](#requirements-addressed)
- [Setup](#setup)
  - [Install tools and dependencies](#install-tools-and-dependencies)
  - [Register your bot](#register-your-bot)
  - [First deployment](#first-deployment)
  - [Register the bot web hook with Telegram](#register-the-bot-web-hook-with-telegram)
  - [You're done!](#youre-done)
- [Developer's notes](#developers-notes)
  - [Project structure](#project-structure)
  - [App code base](#app-code-base)
    - [`bot.js`](#botjs)
    - [`config.js`](#configjs)
  - [Web presentation](#web-presentation)
  - [App deployment setup](#app-deployment-setup)
    - [`.vercelignore`](#vercelignore)
    - [`vercel.json`](#verceljson)
    - [`package.json`](#packagejson)
    - [`setup.sh`](#setupsh)
- [Credits](#credits)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->
<!-- generated with [DocToc](https://github.com/thlorenz/doctoc) -->

## Requirements Addressed

As a user I want to
- pass the quiz
- get assessment of my skills
- retake the quiz later to assess my progress

As a service owner I want to
- keep track of user's responses to show progress

## Setup

This app is developed to run on [`vercel`](https://vercel.com/docs) platform.

Installation workflow:
1. Install tools and dependencies
1. Register the bot and store Telegram API access token
1. Register the database instance
1. Deploy bot as a lambda on `vercel`
1. Register bot webhook with Telegram 

The below instructions are valid as of May 2022

### Install tools and dependencies

You are expected to have the following installed:
- NodeJS and npm.
  [NodeJS v14.19.1](https://nodejs.org/en/blog/release/v14.19.1/)
  is recommended (`vercel` hosting uses NodeJS v12.x, v14.x)
- [yarn](https://yarnpkg.com/en/docs/install) package manager 
  (employed by `vercel`)

<details><summary>Note on NodeJS</summary>
<p>

It is recommended to use [nvm](https://github.com/nvm-sh/nvm/releases)
or [nvm for Windows](https://github.com/coreybutler/nvm-windows)
to manage NodeJS versions.

Under Windows open `cmd.exe` or `PowerShell` as administrator to use `nvm`.
</p>
</details>

Set up account on [vercel](https://vercel.com/). Integration with
GitHub is optional.

`./setup.sh install` will install dependencies and `vercel` CLI

You may need `sudo` installation.

Once `vercel` CLI is installed it is recommended to login
from your current device (normally it is done once for
all projects supported with `vercel`).

`./setup.sh login <your_email_associated_with_vercel_account>`

### Register your bot

Register your bot with [BotFather](https://telegram.me/botfather)
and take a note of API access token.

Store your token with `vercel` secrets storage.

`.setup.sh save_token <telegram_api_token>`

Now your token is securely stored and will be
- available at every device you have logged into `vercel` from
- supplied by `vercel` to the app at app execution as an env variable

You may list stored secrets with `vercel secrets ls`.
You will not be able to retrieve the raw content of your secrets.
However, you always can consult [BotFather](https://telegram.me/botfather) 
to retrieve the token.

It is recommended that you store all secrets like credentials,
access tokens etc with the secrets storage, not as some plain text in
your code base.

### First deployment

`./setup.sh deploy`

Take a note of the complete url of your end-point alias.
Example: `https://app-name-your-vercel-username.vercel.app`

### Register the bot web hook with Telegram

Telegram needs to know where to send messages addressed to
your bot.

This end-point is called a web hook.

`./setup.sh set_webhook <your_project_url.vercel.app> <telegram_api_token>`

### You're done!

You may want change something in your bot code.
Once you're ready to deploy just run `vercel`.

Your projects are accessible via [dashboard](https://vercel.com/dashboard). 
Each project lists all deployments. You may want to
delete obsolete deployments. You will also find logs useful. 

## Developer's notes

### Project structure

```
-\
 |-- api                   ## app code base
 |   |-- bot.js            ## entry point / lambda function
 |   |-- config.js         ## app config
 |   |-- quiz.js           ## quiz model
 |   \-- store.js          ## store module
 |
 |-- assets
 |   \-- tg-logo.png  ## image used as bot's avatar in Telegram
 |
 |-- www                     ## web presentation
 |   |-- default-beauty.css  ## basic styling
 |   \-- index.html          ## default index.html
 |
 |-- .gitignore
 |-- .vercelignore    ## files to be ignore by vercel
 |-- LICENSE.md
 |-- vercel.json      ## deployment setup
 |-- package.json     ## project properties
 |-- README.md
 |-- setup.sh         ## setup script
 \-- yarn.lock
```

### App code base

#### `bot.js`

`vercel` requires app entry points to export functions 
that are ready to serve network requests.
Since Telegram API sends messages via POST
and POST request body is not available synchronously
the function exported is async and `micro` is employed
to process the request properly.

[vercel examples](https://vercel.com/docs/serverless-functions/supported-languages#node.js)
demonstrate GET requests processing, which are synchronous.

* `bot` (exported) - 
  an async function to serve requests
* `emulateConsoleInput` - a method to echo user input
  emulating console input; in bigger chats it may happen
  that some users may post before bot responds so
  echoing is helpful
* `sendMessage` - a method to send response using Telegram
  API

The project uses is tested under
[NodeJS v14.19.1](https://nodejs.org/en/blog/release/v14.19.1/)
that is one of NodeJS versions
[used by vercel](https://vercel.com/docs/runtimes#official-runtimes/node-js)
as of writing this docs.

#### `config.js`

* `telegramApiUrl` - [Telegram API](https://core.telegram.org/bots/api)
  v5.0 url to send responses
  back to client; used in `bot.js`

### Web presentation

These files are served whenever a user access the app via
browser at app domain root.

### App deployment setup

#### `.vercelignore`

Files that 
[shouldn't be uploaded to `vercel`](https://vercel.com/guides/prevent-uploading-sourcepaths-with-vercelignore).
Normally those are files that aren't required to build a project
and aren't used in production. 
Syntax is similar to that of `.gitignore`.

#### `vercel.json`

[`vercel` deployment configuration](https://vercel.com/docs/configuration).

> **NB!** Current config uses legacy configuration approaches.
> Should be upgraded to match the approach as documented
> under the link above.

* `version` - platform version (legacy)
* `name` - project name (legacy)
* `builds` - project build settings, defining e.g.
  which files are static and should be served as is,
  and which should be processed by NodeJS before deployment
  (legacy)
* `routes` - routing rules, defining e.g.
  what file to serve when a web user accesses project root
  and how a web hook (access point url) maps to a lambda
  (legacy)
* `env` - env variables, accessible via `process.env` object;
  when value starts with `@` the real value is taken from
  `vercel` secrets storage
  (legacy)

#### `package.json`

Note how `main` property points at your primary lambda function.
It might be also an `index.js` that sets up a local server for local tests. 

#### `setup.sh`

A script used to set the project up. 

## Credits

Telegram botification is inspired by
[How to build a telegram bot using Node.js and Now](https://scotch.io/tutorials/how-to-build-a-telegram-bot-using-nodejs-and-now)
[2018-10-19] and
[How to make a responsive telegram bot ](https://www.sohamkamani.com/blog/2016/09/21/making-a-telegram-bot/)
[2016-09-21].
