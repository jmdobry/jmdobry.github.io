---
layout: post
title: "Introduction to Data Modeling with JSData"
tags: technology, javascript
---
Inspired by [Ember Data](https://github.com/emberjs/data), [__JSData__](http://www.js-data.io) is a framework-agnostic, datastore-agnostic ORM for Node.js and the Browser.

[JSData's adapters](http://www.js-data.io/docs/working-with-adapters) handle communication with various storage layers, such as `localStorage`, Firebase, RethinkDB, or your RESTful backend.

In a typical scenario, you load data into the store, which maintains a single representation of every unique record coming from your storage layer. The data store offers an API for read, update, and delete operations, which are executed in your storage layer by an adapter, with the results finally synced back to the store. This is your _Data_ or _Model layer_.

The Model layer is typically where your business logic resides–where you manipulate your data. There are many variations on this pattern, and JSData can work with your preferences.

JSData runs in the browser, communicating with storage layers such as `localStorage`, Firebase, your RESTful backend (HTTP target), etc.

JSData also runs in NodeJS, where adapters for MongoDB, Redis, RethinkDB, MySql/Postgres/SQLite, etc. are available.

JSData presents a uniform API for executing your typical [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) operations against any storage layer for which an adapter is available. You can easily combine adapters for more complicated data management.

It is easy to add JSData to your project. Let's get started:

#### Install
```
<npm|bower> install --save js-data
```

JSData is available on [cdnjs](https://cdnjs.com/libraries/js-data) and [jsDelivr](http://www.jsdelivr.com/#!js-data).

You can add JSData to your app as a script tag, via `import JSData from 'js-data'`, `require('js-data')`, `define(['js-data'], ...)`, etc.

#### Create a store

<iframe style="width: 100%;" src="http://embed.plnkr.co/H0XBoFdaGI4k6YOf3BW0/store.js"></iframe>

The `DS` constructor function takes an optional `options` object which can be used to override the default settings for your new store. You can also create a store via `JSData.createStore([options])`.

#### Connect to storage

We're not actually working with any storage layers yet, so let's register an adapter.

```
<npm|bower> install --save js-data-firebase
```

js-data-firebase is available on [cdnjs](https://cdnjs.com/libraries/js-data-firebase) and [jsDelivr](http://www.jsdelivr.com/#!js-data-firebase).

You can add js-data-firebase to your app as a script tag, via `import DSFirebaseAdapter from 'js-data-firebase'`, `require('js-data-firebase')`, `define(['js-data', 'js-data-firebase'], ...)`, etc.

<iframe style="width: 100%; height: 235px;" src="http://embed.plnkr.co/H0XBoFdaGI4k6YOf3BW0/register-firebase.js"></iframe>

#### Model your data

You start modeling your data by registering Resources with the store:

<iframe style="width: 100%;" src="http://embed.plnkr.co/H0XBoFdaGI4k6YOf3BW0/User.js"></iframe>

A [JSData Resource](http://www.js-data.io/docs/resources) defines metadata that the store uses to interact with data of that type. The Resource object returned by `DS#defineResource(options)` exposes an API for working directly with data of that Resource.

With a few lines of code, we can already do all kinds of things with data of type `User`.

<iframe style="width: 100%;height:330px" src="http://embed.plnkr.co/H0XBoFdaGI4k6YOf3BW0/inject-user.js"></iframe>

#### Create a user in Firebase

<iframe style="width: 100%;height:350px" src="http://embed.plnkr.co/H0XBoFdaGI4k6YOf3BW0/create.js"></iframe>

#### Update the user

<iframe style="width: 100%;height:540px" src="http://embed.plnkr.co/H0XBoFdaGI4k6YOf3BW0/update.js"></iframe>

#### Destroy the user

<iframe style="width: 100%;height:390px" src="http://embed.plnkr.co/H0XBoFdaGI4k6YOf3BW0/destroy.js"></iframe>

#### The tip of the iceberg

I hardly had to write an code, and I already have a fully functional CRUD API on top of Firebase! I can easily add more resources, setup relations between them, and even add another adapter for a more complex application.

You know when you show a treat to a starving (but obedient) puppy, and the puppy just starts drooling all over the floor? That puppy is you right now.

JSData will save you time. It's the Twitter Bootstrap of data layers.

Knock yourself out:

- [JSData on Github](https://github.com/js-data)
- [Gitter Channel](https://gitter.im/js-data/js-data)
- [Mailing List](https://groups.io/org/groupsio/jsdata)
- [JSData website](http://www.js-data.io/)
- [MtnWestJS 2015 Presentation](https://www.youtube.com/watch?v=8wxnnJA9FKw)
- [Resources/Models](http://www.js-data.io/docs/resources)
- [Working with the Data Store](http://www.js-data.io/docs/working-with-the-data-store)
- [Adapters](http://www.js-data.io/docs/working-with-adapters)
- [Model Lifecycle](http://www.js-data.io/docs/model-lifecycle)
- [Custom Instance Behavior](http://www.js-data.io/docs/custom-instance-behavior)
- [Computed Properties](http://www.js-data.io/docs/computed-properties)
- [Query Syntax](http://www.js-data.io/docs/query-syntax)
- [Relations](http://www.js-data.io/docs/relations)
- [JSData on the Server](http://www.js-data.io/docs/jsdata-on-the-server)