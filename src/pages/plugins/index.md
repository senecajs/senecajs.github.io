---
layout: content.html
---

# Plugins
*Seneca* plugins enable you to build awesome microservices, incredibly fast.
Some plugins are maintained in-house, while others are provided by the community.

On this page, we provide an `npm` link and total downloads per month for any plugins
we find or publish ourselves, so you only need to look in one place.

## Core plugins
Seneca includes a number of **core plugins** by default. It does this in order to ensure that you can create a **working microservice out of the box**.

### [Web](https://npmjs.org/package/seneca-web)
[![version][web-npm-version]][web-npm-url]
[![downloads][web-npm-downloads]][web-npm-url]
[web-npm-version]: https://img.shields.io/npm/v/seneca-web.svg?style=flat-square
[web-npm-downloads]: https://img.shields.io/npm/dm/seneca-web.svg?style=flat-square
[web-npm-url]: https://npmjs.org/package/seneca-web

Provides a **web service API routing layer** for Seneca action patterns. It translates HTTP requests with specific
URL routes into action pattern calls. It's a built-in dependency of the Seneca module, so you don't need to
include it manually.

### [Transport](https://npmjs.org/package/seneca-transport)
[![version][transport-npm-version]][transport-npm-url]
[![downloads][transport-npm-downloads]][transport-npm-url]
[transport-npm-version]: https://img.shields.io/npm/v/seneca-transport.svg?style=flat-square
[transport-npm-downloads]: https://img.shields.io/npm/dm/seneca-transport.svg?style=flat-square
[transport-npm-url]: https://npmjs.org/package/seneca-transport

Provides the **HTTP and TCP transport channels** for microservice messages. It's a *built-in dependency of the Seneca
module, so you don't need to include it manually.

## Feature plugins
Use these plugins to kick-start your application development. They provide the basic business logic for
many common use cases, like user accounts, shopping carts and administration.  You can customize their behavior
by overriding their actions.

### [User](https://npmjs.org/package/seneca-user)
[![version][user-npm-version]][user-npm-url]
[![downloads][user-npm-downloads]][user-npm-url]
[user-npm-version]: https://img.shields.io/npm/v/seneca-user.svg?style=flat-square
[user-npm-downloads]: https://img.shields.io/npm/dm/seneca-user.svg?style=flat-square
[user-npm-url]: https://npmjs.org/package/seneca-user

Provides business logic for **complete user management**, such as login, logout, registration and password
handling, using Seneca's expressive action-based API. This module complements `Auth` (below).

### [Auth](https://npmjs.org/package/seneca-auth)
[![version][auth-npm-version]][auth-npm-url]
[![downloads][auth-npm-downloads]][auth-npm-url]
[auth-npm-version]: https://img.shields.io/npm/v/seneca-auth.svg?style=flat-square
[auth-npm-downloads]: https://img.shields.io/npm/dm/seneca-auth.svg?style=flat-square
[auth-npm-url]: https://npmjs.org/package/seneca-auth

Provides the business logic for **authentication via HTTP**. Adds the ability to set up simple win/fail-style conditions. The User plugin complements this one nicely.

### [JSON REST API](https://npmjs.org/package/seneca-jsonrest-api)
[![version][jsonrest-npm-version]][jsonrest-npm-url]
[![downloads][jsonrest-npm-downloads]][jsonrest-npm-url]
[jsonrest-npm-version]: https://img.shields.io/npm/v/seneca-jsonrest-api.svg?style=flat-square
[jsonrest-npm-downloads]: https://img.shields.io/npm/dm/seneca-jsonrest-api.svg?style=flat-square
[jsonrest-npm-url]: https://npmjs.org/package/seneca-jsonrest-api

Provides the ability to **expose your data entities as a REST API**. Works via pattern matching actions to HTTP
verbs. Removes the need for additional plumbing to expose entities as resources.

### [Data editor](https://npmjs.org/package/seneca-data-editor)
[![version][data-editor-npm-version]][data-editor-npm-url]
[![downloads][data-editor-npm-downloads]][data-editor-npm-url]
[data-editor-npm-version]: https://img.shields.io/npm/v/seneca-data-editor.svg?style=flat-square
[data-editor-npm-downloads]: https://img.shields.io/npm/dm/seneca-data-editor.svg?style=flat-square
[data-editor-npm-url]: https://npmjs.org/package/seneca-data-editor

Provides an **administrative interface** for editing all the data in your system. This plugin can be used alone, but also pairs well with `seneca-admin`. Inspired by the Django admin interface.

### [Admin](https://npmjs.org/package/seneca-admin)
[![version][admin-npm-version]][admin-npm-url]
[![downloads][admin-npm-downloads]][admin-npm-url]
[admin-npm-version]: https://img.shields.io/npm/v/seneca-admin.svg?style=flat-square
[admin-npm-downloads]: https://img.shields.io/npm/dm/seneca-admin.svg?style=flat-square
[admin-npm-url]: https://npmjs.org/package/seneca-admin

Provides an **administration console** that returns interesting data about your microservice, including
streaming log, status summaries and action patterns. Can be locked to admin for safety purposes.

### [Email](https://npmjs.org/package/seneca-mail)
[![version][mail-npm-version]][mail-npm-url]
[![downloads][mail-npm-downloads]][mail-npm-url]
[mail-npm-version]: https://img.shields.io/npm/v/seneca-mail.svg?style=flat-square
[mail-npm-downloads]: https://img.shields.io/npm/dm/seneca-mail.svg?style=flat-square
[mail-npm-url]: https://npmjs.org/package/seneca-mail

Provides **email delivery business logic**, including the ability to have templates. Works with many
different providers. Works well with the User plugin for handling things like welcome mails and
password reminders.

### [Account](https://npmjs.org/package/seneca-account)
[![version][account-npm-version]][account-npm-url]
[![downloads][account-npm-downloads]][account-npm-url]
[account-npm-version]: https://img.shields.io/npm/v/seneca-account.svg?style=flat-square
[account-npm-downloads]: https://img.shields.io/npm/dm/seneca-account.svg?style=flat-square
[account-npm-url]: https://npmjs.org/package/seneca-account

Provides a **user account system** for managing multiple users in an account. Handles account creation
and management, and user management within a given account. The `User` plugin compliments this one nicely.

### [Project](https://npmjs.org/package/seneca-project)
[![version][project-npm-version]][project-npm-url]
[![downloads][project-npm-downloads]][project-npm-url]
[project-npm-version]: https://img.shields.io/npm/v/seneca-project.svg?style=flat-square
[project-npm-downloads]: https://img.shields.io/npm/dm/seneca-project.svg?style=flat-square
[project-npm-url]: https://npmjs.org/package/seneca-project

Provides all the actions needed to **manage a project**. Use this plugin to build out microservices
that have the concepts of ownership, grouping, and starting, stopping and loading of some type of work. Works
well with the `Accounts` plugin.

### [Perm](https://npmjs.org/package/seneca-perm)
[![version][perm-npm-version]][perm-npm-url]
[![downloads][perm-npm-downloads]][perm-npm-url]
[perm-npm-version]: https://img.shields.io/npm/v/seneca-perm.svg?style=flat-square
[perm-npm-downloads]: https://img.shields.io/npm/dm/seneca-perm.svg?style=flat-square
[perm-npm-url]: https://npmjs.org/package/seneca-perm

Provides a **permissions system for actions**. This plugin works by wrapping existing actions with a permission
checking action. If the permission test passes, the parent action can proceed. If not, a permission error
is generated.

### [VCache](https://npmjs.org/package/seneca-vcache)
[![version][vcache-npm-version]][vcache-npm-url]
[![downloads][vcache-npm-downloads]][vcache-npm-url]
[vcache-npm-version]: https://img.shields.io/npm/v/seneca-vcache.svg?style=flat-square
[vcache-npm-downloads]: https://img.shields.io/npm/dm/seneca-vcache.svg?style=flat-square
[vcache-npm-url]: https://npmjs.org/package/seneca-vcache

Provides a **data caching mechanism for data entities**. Using this module will give your Seneca app a big
performance boost. The caching mechanism goes beyond simple key-based caching using `memcached`. In addition,
a smaller "hot" cache is maintained within the Node.js process. Data entities are given transient version numbers
and these are used to synchronize the hot cache with memcached.

### [Cart](https://npmjs.org/package/seneca-cart)
[![version][cart-npm-version]][cart-npm-url]
[![downloads][cart-npm-downloads]][cart-npm-url]
[cart-npm-version]: https://img.shields.io/npm/v/seneca-cart.svg?style=flat-square
[cart-npm-downloads]: https://img.shields.io/npm/dm/seneca-cart.svg?style=flat-square
[cart-npm-url]: https://npmjs.org/package/seneca-cart

Provides complete **shopping cart management business logic**. This plugin works really well with the built-in
data entity API. Adds actions for adding, removing and editing items in a container (cart).

### [Pay](https://npmjs.org/package/seneca-pay)
[![version][pay-npm-version]][pay-npm-url]
[![downloads][pay-npm-downloads]][pay-npm-url]
[pay-npm-version]: https://img.shields.io/npm/v/seneca-pay.svg?style=flat-square
[pay-npm-downloads]: https://img.shields.io/npm/dm/seneca-pay.svg?style=flat-square
[pay-npm-url]: https://npmjs.org/package/seneca-pay

Provides all the necessary components to build **payments** into your microservice. Includes support for Paypal
express payments in the box. Makes setting up payment redirects a breeze.

### [CMS](https://npmjs.org/package/seneca-cms)
[![version][cms-npm-version]][cms-npm-url]
[![downloads][cms-npm-downloads]][cms-npm-url]
[cms-npm-version]: https://img.shields.io/npm/v/seneca-cms.svg?style=flat-square
[cms-npm-downloads]: https://img.shields.io/npm/dm/seneca-cms.svg?style=flat-square
[cms-npm-url]: https://npmjs.org/package/seneca-cms

Provides a **simple content management system**. Use it to create a more specialized system that fits your needs.
Handles the management of a system's unique entities in a more coarse fashion than using the Entity API
directly (which is used internally by this plugin).

### [Settings](https://npmjs.org/package/seneca-settings)
[![version][settings-npm-version]][settings-npm-url]
[![downloads][settings-npm-downloads]][settings-npm-url]
[settings-npm-version]: https://img.shields.io/npm/v/seneca-settings.svg?style=flat-square
[settings-npm-downloads]: https://img.shields.io/npm/dm/seneca-settings.svg?style=flat-square
[settings-npm-url]: https://npmjs.org/package/seneca-settings

**Rich settings for user accounts**. Can handle many different types of values including ratings,
toggles, colors and ranges.

## Storage plugins
Storage plugins work with our built-in [Entity API](). Storage plugins can be used on a per-entity
basis, so feel free to mix and match. Each plugin is named after the storage it supports, making it simple to find
the right solution.

### [Mongo store](https://npmjs.org/package/seneca-mongo-store)
[![version][mongo-store-npm-version]][mongo-store-npm-url]
[![downloads][mongo-store-npm-downloads]][mongo-store-npm-url]
[mongo-store-npm-version]: https://img.shields.io/npm/v/seneca-mongo-store.svg?style=flat-square
[mongo-store-npm-downloads]: https://img.shields.io/npm/dm/seneca-mongo-store.svg?style=flat-square
[mongo-store-npm-url]: https://npmjs.org/package/seneca-mongo-store

### [Postgres store](https://npmjs.org/package/seneca-postgres-store)
[![version][postgres-store-npm-version]][postgres-store-npm-url]
[![downloads][postgres-store-npm-downloads]][postgres-store-npm-url]
[postgres-store-npm-version]: https://img.shields.io/npm/v/seneca-postgres-store.svg?style=flat-square
[postgres-store-npm-downloads]: https://img.shields.io/npm/dm/seneca-postgres-store.svg?style=flat-square
[postgres-store-npm-url]: https://npmjs.org/package/seneca-postgres-store

### [MySQL store](https://npmjs.org/package/seneca-mysql-store)
[![version][mysql-store-npm-version]][mysql-store-npm-url]
[![downloads][mysql-store-npm-downloads]][mysql-store-npm-url]
[mysql-store-npm-version]: https://img.shields.io/npm/v/seneca-mysql-store.svg?style=flat-square
[mysql-store-npm-downloads]: https://img.shields.io/npm/dm/seneca-mysql-store.svg?style=flat-square
[mysql-store-npm-url]: https://npmjs.org/package/seneca-mysql-store

### [Level store](https://npmjs.org/package/seneca-level-store)
[![version][level-store-npm-version]][level-store-npm-url]
[![downloads][level-store-npm-downloads]][level-store-npm-url]
[level-store-npm-version]: https://img.shields.io/npm/v/seneca-level-store.svg?style=flat-square
[level-store-npm-downloads]: https://img.shields.io/npm/dm/seneca-level-store.svg?style=flat-square
[level-store-npm-url]: https://npmjs.org/package/seneca-level-store

### [JSON file store](https://npmjs.org/package/seneca-jsonfile-store)
[![version][jsonfile-store-npm-version]][jsonfile-store-npm-url]
[![downloads][jsonfile-store-npm-downloads]][jsonfile-store-npm-url]
[jsonfile-store-npm-version]: https://img.shields.io/npm/v/seneca-jsonfile-store.svg?style=flat-square
[jsonfile-store-npm-downloads]: https://img.shields.io/npm/dm/seneca-jsonfile-store.svg?style=flat-square
[jsonfile-store-npm-url]: https://npmjs.org/package/seneca-jsonfile-store

### [Redis store](https://npmjs.org/package/seneca-redis-store)
[![version][redis-store-npm-version]][redis-store-npm-url]
[![downloads][redis-store-npm-downloads]][redis-store-npm-url]
[redis-store-npm-version]: https://img.shields.io/npm/v/seneca-redis-store.svg?style=flat-square
[redis-store-npm-downloads]: https://img.shields.io/npm/dm/seneca-redis-store.svg?style=flat-square
[redis-store-npm-url]: https://npmjs.org/package/seneca-redis-store

### [Dynamo store](https://npmjs.org/package/seneca-dynamo-store)
[![version][dynamo-store-npm-version]][dynamo-store-npm-url]
[![downloads][dynamo-store-npm-downloads]][dynamo-store-npm-url]
[dynamo-store-npm-version]: https://img.shields.io/npm/v/seneca-dynamo-store.svg?style=flat-square
[dynamo-store-npm-downloads]: https://img.shields.io/npm/dm/seneca-dynamo-store.svg?style=flat-square
[dynamo-store-npm-url]: https://npmjs.org/package/seneca-dynamo-store

### [HANA store](https://npmjs.org/package/seneca-hana-store)
[![version][hana-store-npm-version]][hana-store-npm-url]
[![downloads][hana-store-npm-downloads]][hana-store-npm-url]
[hana-store-npm-version]: https://img.shields.io/npm/v/seneca-hana-store.svg?style=flat-square
[hana-store-npm-downloads]: https://img.shields.io/npm/dm/seneca-hana-store.svg?style=flat-square
[hana-store-npm-url]: https://npmjs.org/package/seneca-hana-store

### [SQLite store](https://npmjs.org/package/seneca-sqlite-store)
[![version][sqlite-store-npm-version]][sqlite-store-npm-url]
[![downloads][sqlite-store-npm-downloads]][sqlite-store-npm-url]
[sqlite-store-npm-version]: https://img.shields.io/npm/v/seneca-sqlite-store.svg?style=flat-square
[sqlite-store-npm-downloads]: https://img.shields.io/npm/dm/seneca-sqlite-store.svg?style=flat-square
[sqlite-store-npm-url]: https://npmjs.org/package/seneca-sqlite-store

### [Riak store](https://npmjs.org/package/seneca-riak-store)
[![version][riak-store-npm-version]][riak-store-npm-url]
[![downloads][riak-store-npm-downloads]][riak-store-npm-url]
[riak-store-npm-version]: https://img.shields.io/npm/v/seneca-riak-store.svg?style=flat-square
[riak-store-npm-downloads]: https://img.shields.io/npm/dm/seneca-riak-store.svg?style=flat-square
[riak-store-npm-url]: https://npmjs.org/package/seneca-riak-store

### [Cassandra store](https://npmjs.org/package/seneca-cassandra-store)
[![version][cassandra-store-npm-version]][cassandra-store-npm-url]
[![downloads][cassandra-store-npm-downloads]][cassandra-store-npm-url]
[cassandra-store-npm-version]: https://img.shields.io/npm/v/seneca-cassandra-store.svg?style=flat-square
[cassandra-store-npm-downloads]: https://img.shields.io/npm/dm/seneca-cassandra-store.svg?style=flat-square
[cassandra-store-npm-url]: https://npmjs.org/package/seneca-cassandra-store

### [CouchDB store](https://npmjs.org/package/seneca-couchdb-store)
[![version][couchdb-store-npm-version]][couchdb-store-npm-url]
[![downloads][couchdb-store-npm-downloads]][couchdb-store-npm-url]
[couchdb-store-npm-version]: https://img.shields.io/npm/v/seneca-couchdb-store.svg?style=flat-square
[couchdb-store-npm-downloads]: https://img.shields.io/npm/dm/seneca-couchdb-store.svg?style=flat-square
[couchdb-store-npm-url]: https://npmjs.org/package/seneca-couchdb-store

### [CouchDB version 2 store](https://npmjs.org/package/seneca-couch2-store)
[![version][couch2-store-npm-version]][couch2-store-npm-url]
[![downloads][couch2-store-npm-downloads]][couch2-store-npm-url]
[couch2-store-npm-version]: https://img.shields.io/npm/v/seneca-couch2-store.svg?style=flat-square
[couch2-store-npm-downloads]: https://img.shields.io/npm/dm/seneca-couch2-store.svg?style=flat-square
[couch2-store-npm-url]: https://npmjs.org/package/seneca-couch2-store

### [SimpleDB store](https://npmjs.org/package/seneca-simpledb-store)
[![version][simpledb-store-npm-version]][simpledb-store-npm-url]
[![downloads][simpledb-store-npm-downloads]][simpledb-store-npm-url]
[simpledb-store-npm-version]: https://img.shields.io/npm/v/seneca-simpledb-store.svg?style=flat-square
[simpledb-store-npm-downloads]: https://img.shields.io/npm/dm/seneca-simpledb-store.svg?style=flat-square
[simpledb-store-npm-url]: https://npmjs.org/package/seneca-simpledb-store

## Transport plugins
Seneca supports many different transports. Use the one that fits best with your project right now; swap it out if you scale. Each plugin is named after the transport it supports, making it easy to find the right solution.
### [Transport](https://npmjs.com/package/seneca-transport)
[![version][seneca-transport-npm-version]][seneca-transport-npm-url]
[![downloads][seneca-transport-npm-downloads]][seneca-transport-npm-url]
[seneca-transport-npm-version]: https://img.shields.io/npm/v/seneca-transport.svg?style=flat-square
[seneca-transport-npm-downloads]: https://img.shields.io/npm/dm/seneca-transport.svg?style=flat-square
[seneca-transport-npm-url]: https://npmjs.org/package/seneca-transport

### [AMQP](https://npmjs.com/package/seneca-amqp-transport)
[![version][seneca-amqp-transport-npm-version]][seneca-amqp-transport-npm-url]
[![downloads][seneca-amqp-transport-npm-downloads]][seneca-amqp-transport-npm-url]
[seneca-amqp-transport-npm-version]: https://img.shields.io/npm/v/seneca-amqp-transport.svg?style=flat-square
[seneca-amqp-transport-npm-downloads]: https://img.shields.io/npm/dm/seneca-amqp-transport.svg?style=flat-square
[seneca-amqp-transport-npm-url]: https://npmjs.org/package/seneca-amqp-transport

### [Kafka](https://npmjs.com/package/seneca-kafka-transport)
[![version][seneca-kafka-transport-npm-version]][seneca-kafka-transport-npm-url]
[![downloads][seneca-kafka-transport-npm-downloads]][seneca-kafka-transport-npm-url]
[seneca-kafka-transport-npm-version]: https://img.shields.io/npm/v/seneca-kafka-transport.svg?style=flat-square
[seneca-kafka-transport-npm-downloads]: https://img.shields.io/npm/dm/seneca-kafka-transport.svg?style=flat-square
[seneca-kafka-transport-npm-url]: https://npmjs.org/package/seneca-kafka-transport


### [Redis](https://npmjs.com/package/seneca-redis-transport)
[![version][seneca-redis-transport-npm-version]][seneca-redis-transport-npm-url]
[![downloads][seneca-redis-transport-npm-downloads]][seneca-redis-transport-npm-url]
[seneca-redis-transport-npm-version]: https://img.shields.io/npm/v/seneca-redis-transport.svg?style=flat-square
[seneca-redis-transport-npm-downloads]: https://img.shields.io/npm/dm/seneca-redis-transport.svg?style=flat-square
[seneca-redis-transport-npm-url]: https://npmjs.org/package/seneca-redis-transport

### [Load-Balance](https://npmjs.com/package/seneca-loadbalance-transport)
[![version][seneca-loadbalance-transport-npm-version]][seneca-loadbalance-transport-npm-url]
[![downloads][seneca-loadbalance-transport-npm-downloads]][seneca-loadbalance-transport-npm-url]
[seneca-loadbalance-transport-npm-version]: https://img.shields.io/npm/v/seneca-loadbalance-transport.svg?style=flat-square
[seneca-loadbalance-transport-npm-downloads]: https://img.shields.io/npm/dm/seneca-loadbalance-transport.svg?style=flat-square
[seneca-loadbalance-transport-npm-url]: https://npmjs.org/package/seneca-loadbalance-transport

### [Beanstalk](https://npmjs.com/package/seneca-beanstalk-transport)
[![version][seneca-beanstalk-transport-npm-version]][seneca-beanstalk-transport-npm-url]
[![downloads][seneca-beanstalk-transport-npm-downloads]][seneca-beanstalk-transport-npm-url]
[seneca-beanstalk-transport-npm-version]: https://img.shields.io/npm/v/seneca-beanstalk-transport.svg?style=flat-square
[seneca-beanstalk-transport-npm-downloads]: https://img.shields.io/npm/dm/seneca-beanstalk-transport.svg?style=flat-square
[seneca-beanstalk-transport-npm-url]: https://npmjs.org/package/seneca-beanstalk-transport

### [Queue](https://npmjs.com/package/seneca-queue)
[![version][seneca-queue-npm-version]][seneca-queue-npm-url]
[![downloads][seneca-queue-npm-downloads]][seneca-queue-npm-url]
[seneca-queue-npm-version]: https://img.shields.io/npm/v/seneca-queue.svg?style=flat-square
[seneca-queue-npm-downloads]: https://img.shields.io/npm/dm/seneca-queue.svg?style=flat-square
[seneca-queue-npm-url]: https://npmjs.org/package/seneca-queue

### [ActiveMQ](https://npmjs.com/package/seneca-activemq-transport)
[![version][seneca-activemq-transport-npm-version]][seneca-activemq-transport-npm-url]
[![downloads][seneca-activemq-transport-npm-downloads]][seneca-activemq-transport-npm-url]
[seneca-activemq-transport-npm-version]: https://img.shields.io/npm/v/seneca-activemq-transport.svg?style=flat-square
[seneca-activemq-transport-npm-downloads]: https://img.shields.io/npm/dm/seneca-activemq-transport.svg?style=flat-square
[seneca-activemq-transport-npm-url]: https://npmjs.org/package/seneca-activemq-transport

### [Redis Queue](https://npmjs.com/package/seneca-redis-queue-transport)
[![version][seneca-redis-queue-transport-npm-version]][seneca-redis-queue-transport-npm-url]
[![downloads][seneca-redis-queue-transport-npm-downloads]][seneca-redis-queue-transport-npm-url]
[seneca-redis-queue-transport-npm-version]: https://img.shields.io/npm/v/seneca-redis-queue-transport.svg?style=flat-square
[seneca-redis-queue-transport-npm-downloads]: https://img.shields.io/npm/dm/seneca-redis-queue-transport.svg?style=flat-square
[seneca-redis-queue-transport-npm-url]: https://npmjs.org/package/seneca-redis-queue-transport

### [NATS](https://npmjs.com/package/seneca-nats-transport)
[![version][seneca-nats-transport-npm-version]][seneca-nats-transport-npm-url]
[![downloads][seneca-nats-transport-npm-downloads]][seneca-nats-transport-npm-url]
[seneca-nats-transport-npm-version]: https://img.shields.io/npm/v/seneca-nats-transport.svg?style=flat-square
[seneca-nats-transport-npm-downloads]: https://img.shields.io/npm/dm/seneca-nats-transport.svg?style=flat-square
[seneca-nats-transport-npm-url]: https://npmjs.org/package/seneca-nats-transport

### [NSQ](https://npmjs.com/package/seneca-nsq-transport)
[![version][seneca-nsq-transport-npm-version]][seneca-nsq-transport-npm-url]
[![downloads][seneca-nsq-transport-npm-downloads]][seneca-nsq-transport-npm-url]
[seneca-nsq-transport-npm-version]: https://img.shields.io/npm/v/seneca-nsq-transport.svg?style=flat-square
[seneca-nsq-transport-npm-downloads]: https://img.shields.io/npm/dm/seneca-nsq-transport.svg?style=flat-square
[seneca-nsq-transport-npm-url]: https://npmjs.org/package/seneca-nsq-transport

### [Stomp](https://npmjs.com/package/seneca-stomp-transport)
[![version][seneca-stomp-transport-npm-version]][seneca-stomp-transport-npm-url]
[![downloads][seneca-stomp-transport-npm-downloads]][seneca-stomp-transport-npm-url]
[seneca-stomp-transport-npm-version]: https://img.shields.io/npm/v/seneca-stomp-transport.svg?style=flat-square
[seneca-stomp-transport-npm-downloads]: https://img.shields.io/npm/dm/seneca-stomp-transport.svg?style=flat-square
[seneca-stomp-transport-npm-url]: https://npmjs.org/package/seneca-stomp-transport

### [Redis Sync](https://npmjs.com/package/seneca-redis-sync-transport)
[![version][seneca-redis-sync-transport-npm-version]][seneca-redis-sync-transport-npm-url]
[![downloads][seneca-redis-sync-transport-npm-downloads]][seneca-redis-sync-transport-npm-url]
[seneca-redis-sync-transport-npm-version]: https://img.shields.io/npm/v/seneca-redis-sync-transport.svg?style=flat-square
[seneca-redis-sync-transport-npm-downloads]: https://img.shields.io/npm/dm/seneca-redis-sync-transport.svg?style=flat-square
[seneca-redis-sync-transport-npm-url]: https://npmjs.org/package/seneca-redis-sync-transport

### [MQ Light](https://npmjs.com/package/seneca-mqlight-transport)
[![version][seneca-mqlight-transport-npm-version]][seneca-mqlight-transport-npm-url]
[![downloads][seneca-mqlight-transport-npm-downloads]][seneca-mqlight-transport-npm-url]
[seneca-mqlight-transport-npm-version]: https://img.shields.io/npm/v/seneca-mqlight-transport.svg?style=flat-square
[seneca-mqlight-transport-npm-downloads]: https://img.shields.io/npm/dm/seneca-mqlight-transport.svg?style=flat-square
[seneca-mqlight-transport-npm-url]: https://npmjs.org/package/seneca-mqlight-transport


## Got a plugin to share?
Our docs are open source. Simply fork the [senecajs.org] repository, add your plugin to the appropriate section,
and send us on a pull request - that way, other people can find your awesome plugin easily.

[senecajs.org]: https://github.com/senecajs/senecajs.org
