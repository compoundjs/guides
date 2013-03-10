## Crash Course to CompoundJS

In this guide we will start with creating app using generators. Goal: get
working application without any additional knoweledge about express, learn
structure and tools coming with railway, quick overview of main features.

### What is compound

Before we start I have define compound framework. Compound's formula:
[Express][express] + structure + extensions. Where **structure** is standard
layout of directories, and **extensions** are node modules adding functionality to
the framework. Compound's goal: provide obvious and well-organized interface for
express application development. That means everything working with express will
work with compound too.

Now we can start building app. Let's create todo-list app with REST API and web
interface.

### First steps: install compound and generate app

1. install compound using npm

    npm install compound

2. generate app

    compound init todo-list-app

3. install dependencies

    cd todo-list app && npm install

Now we have initial compound app structure, we can run application and see
what's happen. Let's run `node .` command and open http://localhost:3000/ in
browser.

> **Note** `node .` command is simple way to run application in current
> directory. You also can run `node server.js` or `coffee server.coffee` or even
> `compound server`.
