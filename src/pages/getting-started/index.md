---
layout: content.html
---
# Getting started
Welcome to the Seneca _Getting started_ guide! This guide assumes that you are already familiar
with [Node.js][] and how to build simple applications with it.

## What is Seneca?
Seneca lets you build message based microservice systems with ease. You don't need to know where
the other services are located, how many of them there are, or what they do. Everything external to
your business logic - such as databases, caches and third-party integrations - is likewise hidden
behind microservices.

This decoupling makes your system easy to continuously build and change. It works because Seneca
has the following three core features:

- __Pattern matching__: Instead of fragile service discovery, you just let the world know what sort of messages you care about.
- __Transport independence__: You can send messages between services in many ways, all hidden from your business logic.
- __Componentisation__: Functionality is expressed as a set of plugins which can be composed together as microservices.

Messages are JSON objects. They can have any internal structure you like. Messages can be sent via
HTTP/S, TCP, message queues, publish/subscribe services or any mechanism that moves bits around.
From your perspective as the writer of a service, you just send messages out into the world. You
don't need to know which services receive them.

Then there are the messages you'd like to receive. You specify the property patterns that you care
about, and Seneca (with a little configuration help) makes sure that you get any messages sent by
other services that match those patterns. The patterns are very simple: just a list of key-value
pairs that must match the top-level properties of the JSON message.

This guide will walk you through seneca principles and teach you how to build microservices with it.

Let's build some microservices!

## Patterns
Let's start with some code. We will create two microservices, one that will do math operations and
another that makes use

``` js
var seneca = require('seneca')()

seneca.add('role:math,cmd:sum', (msg, reply) => {
  reply(null, {answer: (msg.left + msg.right)})
})

seneca.act({role: 'math', cmd: 'sum', left: 1, right: 2}, function (err, result) {
  if (err) return console.error(err)
  console.log(result)
})
```


For the moment, this is all happening in the same process, without network traffic. In-process
function calls are a type of message transport too!

The `seneca.add` method adds a new action pattern to the Seneca instance. It has two parameters:

* `pattern`: the property pattern to match in any JSON messages that the Seneca instance receives.
* `action`: the function to execute when a pattern matches a message.

The action function has two parameters:

* `msg`: the matching inbound message (provided as a plain object).
* `respond`: a callback function that you use to provide a response to the message.

The respond function is a callback with the standard `error, result` signature.

Let's put this all together again:

``` js
seneca.add({role: 'math', cmd: 'sum'}, function (msg, respond) {
  var sum = msg.left + msg.right
  respond(null, {answer: sum})
})
```

In the sample code, the action computes the sum of two numbers, provided via the `left` and
`right` properties of the message object. Not all messages generate a result, but as this is the
most common case, Seneca allows you to provide the result via a callback function.

In summary, the action pattern `role:math,cmd:sum` acts on this message:

``` js
{role: 'math', cmd: 'sum', left: 1, right: 2}
```

to produce this result:

``` js
{answer: 3}
```

There is nothing special about the properties `role` and `cmd`. They just happen to be the ones you
are using for pattern matching.

The `seneca.act` method submits a message to act on. It has two parameters:

* `msg`: the message object.
* `response_callback`: a function that receives the message response, if any.

The response callback is a function you provide with the standard `error, result` signature. If
there is a problem (say, the message matches no patterns), then the first argument is an
[Error][] object. If everything goes according to plan, the second argument is the result object. In the
sample code, these arguments are simply printed to the console:

``` js
seneca.act({role: 'math', cmd: 'sum', left: 1, right: 2}, function (err, result) {
  if (err) return console.error(err)
  console.log(result)
})
```

The sample code in the [sum.js][] file shows you how to define and call an action pattern inside
the same Node.js process. You'll soon see how to split this code over multiple processes.

## How patterns work
Patterns - as opposed to network addresses or topics - make it much easier to extend and enhance your system. They do this by letting you incrementally add new microservices.

Let's add to our system the ability to multiply two numbers.

We want messages that look like this:

``` js
{role: 'math', cmd: 'product', left: 3, right: 4}
```

to produce results like this:

``` js
{answer: 12}
```

You can use the `role: math, cmd: sum` action pattern as a template to define a new
`role: math, cmd: product` action:

``` js
seneca.add({role: 'math', cmd: 'product'}, function (msg, respond) {
  var product = msg.left * msg.right
  respond(null, { answer: product })
})
```

And you can call it in exactly the same way:

``` js
seneca.act({role: 'math', cmd: 'product', left: 3, right: 4}, console.log)
```

Here, you use `console.log` as a shortcut to print out both the error (if any) and the result.
Running this code produces:

``` js
{answer: 12}
```

Putting this all together, you get:

``` js
var seneca = require('seneca')()

seneca.add({role: 'math', cmd: 'sum'}, function (msg, respond) {
  var sum = msg.left + msg.right
  respond(null, {answer: sum})
})

seneca.add({role: 'math', cmd: 'product'}, function (msg, respond) {
  var product = msg.left * msg.right
  respond(null, { answer: product })
})


seneca.act({role: 'math', cmd: 'sum', left: 1, right: 2}, console.log)
      .act({role: 'math', cmd: 'product', left: 3, right: 4}, console.log)
```

In the above code sample, the `seneca.act` calls are chained together. Seneca provides a chaining API as a
convenience. Chained calls are executed in order, but not in series, so their results could come
back in any order.

## Extending functionality by adding patterns
Patterns make it easy for you to extend your functionality. Instead of adding if statements and complex
logic, you simply add more patterns.

Let's extend the addition action by adding the ability to force integer-only arithmetic. To do this, you add a new property, _integer:true_, to the message object. Then, you provide a new action for messages that have this property:

``` js
seneca.add({role: 'math', cmd: 'sum', integer: true}, function (msg, respond) {
  var sum = Math.floor(msg.left) + Math.floor(msg.right)
  respond(null, {answer: sum})
})
```

Now, this message

``` js
{role: 'math', cmd: 'sum', left: 1.5, right: 2.5, integer: true}
```

produces this result:

``` js
{answer: 3}  // == 1 + 2, as decimals removed
```

What happens if you add both patterns to the same system? How does Seneca choose which one to use?
The more specific pattern always wins. In other words, the pattern with the highest number of matching attributes has precedence.

Here's some code to illustrate this:

``` js
var seneca = require('seneca')()

seneca.add({role: 'math', cmd: 'sum'}, function (msg, respond) {
  var sum = msg.left + msg.right
  respond(null, {answer: sum})
})

// both these messages match role: math, cmd: sum


seneca.act({role: 'math', cmd: 'sum', left: 1.5, right: 2.5}, console.log)
seneca.act({role: 'math', cmd: 'sum', left: 1.5, right: 2.5, integer: true}, console.log)

seneca.add({role: 'math', cmd: 'sum', integer: true}, function (msg, respond) {
  var sum = Math.floor(msg.left) + Math.floor(msg.right)
  respond(null, { answer: sum })
})

// this still matches role: math, cmd: sum
seneca.act({role: 'math', cmd: 'sum', left: 1.5, right: 2.5}, console.log)

// BUT this matches role:math,cmd:sum,integer:true
// because it's more specific - more properties match
seneca.act({role: 'math', cmd: 'sum', left: 1.5, right: 2.5, integer: true}, console.log)
```

And the output it generates is:

``` js
2016  ...  INFO  hello  ...
null { answer: 4 }
null { answer: 4 }
null { answer: 4 }
null { answer: 3 }
```

The first two `.act` calls both match the `role: math, cmd: sum` action pattern. Next, the code defines the integer-only action pattern `role: math, cmd: sum, integer: true`. After that, the third call to
`.act` goes with the `role: math, cmd: sum` action, but the fourth goes with `role: math, cmd: sum,
integer: true`. This code also demonstrates that you can chain `.add` and `.act` calls together.
This code is available in the [sum-integer.js][] file.

The ability to easily extend the behavior of your actions by matching more specific kinds of
messages is an easy way to handle new and changing requirements. This applies both while your project is in
development and when it is live and needs to adapt. It also has the advantage that you do not need
to modify existing code. It's much safer to add new code to handle special cases. In a production system, you won't even need to do a re-deploy. Your existing services can stay running as they are. All you need to do is start up your new service.

## Using patterns for code re-use
Action patterns can call other action patterns to do their work. Let's modify our sample
code to use this approach:

``` js
var seneca = require('seneca')()

seneca.add('role: math, cmd: sum', function (msg, respond) {
  var sum = msg.left + msg.right
  respond(null, {answer: sum})
})

seneca.add('role: math, cmd: sum, integer: true', function (msg, respond) {
  // reuse role:math, cmd:sum
  this.act({
    role: 'math',
    cmd: 'sum',
    left: Math.floor(msg.left),
    right: Math.floor(msg.right)
  }, respond)
})

// this matches role:math,cmd:sum
seneca.act('role: math, cmd: sum, left: 1.5, right: 2.5',console.log)

// BUT this matches role:math,cmd:sum,integer:true
seneca.act('role: math, cmd: sum, left: 1.5, right: 2.5, integer: true', console.log)
```

In this version of the code, the definition of the `role: math, cmd: sum, integer: true` action pattern uses
the previously defined `role: math, cmd: sum` action pattern. However, it first modifies the message to
convert the `left` and `right` properties to integers.

Inside the action function, the context variable `this` is a reference to the current Seneca
instance. This is the proper way to reference Seneca inside actions, as you get the full
context of the current action call. This makes your logs more informative, among other things.

This code uses an abbreviated form of JSON to specify the patterns and messages. For example, the
object literal form

``` js
{role: 'math', cmd: 'sum', left: 1.5, right: 2.5}
```

becomes:

``` js
'role: math, cmd: sum, left: 1.5, right: 2.5'
```

This format, [jsonic][], which you provide as a string literal, is a convenient format for making
patterns and messages more concise in your code.

The code for the sample above is available in the [sum-reuse.js][] file.

## Patterns are unique
The action patterns that you define are **unique**. They can trigger only one function. The patterns
resolve using the following rules:

* More properties win.
* If the patterns have the same number of properties, they are matched in alphabetical order.

These rules are designed to be simple so that you can run them in your head. It's very
easy to understand which pattern will trigger which action function.

Here are some examples:

* `a:1, b:2` wins over `a:1` as it has more properties.
* `a:1, b:2` wins over `a:1, c:3` as `b` comes before `c` alphabetically.
* `a:1, b:2, d:4` wins over `a:1, c:3, d:4` as `b` comes before `c` alphabetically.
* `a:1, b:2, c:3` wins over `a:1, b:2` as it has more properties.
* `a:1, b:2, c:3` wins over `a:1, c:3` as it has more properties.

To see this in action, run the file [pattern-wins.js][]. _For more details, see the [patrun module][]_.

It is sometimes useful to have a way of _enhancing the behavior of an action without rewriting it
fully_. For example, you might want to perform custom validation of the message properties,
capture message statistics, add additional information to action results, or throttle message
flow rates.

In the sample code, the addition action expects the `left` and `right` properties to be finite
numbers. Also, it's useful to include the original input arguments in the output for debugging
purposes. You can add a validation check and debugging information using the following code:

``` js
var seneca = require('seneca')()

seneca
  .add(
    'role:math,cmd:sum',
    function (msg, respond) {
      var sum = msg.left + msg.right
      respond(null, { answer: sum })
    })

  // override role:math,cmd:sum with additional functionality
  .add(
    'role:math,cmd:sum',
    function (msg, respond) {

      // bail out early if there's a problem
      if (!Number.isFinite(msg.left) ||
          !Number.isFinite(msg.right))
      {
        return respond(new Error("Expected left and right to be numbers."))
      }

      // call previous action function for role:math,cmd:sum
      this.prior({
        role:  'math',
        cmd:   'sum',
        left:  msg.left,
        right: msg.right,

      }, function (err, result) {
        if (err) return respond(err)

        result.info = msg.left+'+'+msg.right
        respond(null, result)
      })
    })

  // enhanced role:math,cmd:sum
  .act('role:math,cmd:sum,left:1.5,right:2.5',
        console.log // prints { answer: 4, info: '1.5+2.5' }
     )
```

The Seneca instance provided to an action function via the `this` context variable has a special `prior` method that calls the previous action definition for the current action pattern.

The prior function has parameters:

* `msg`: the msg object, which you may have modified.
* `response_callback`: a callback function where you can modify the result.

The example code shows you how to modify both the inbound message and the outbound result. Modification of either is optional. You may leave the data unchanged and use this mechanism for enhanced logging or auditing.

The example code also shows good practice for error handling. It uses early returns to exit from the action function as soon as possible. This avoids spurious indentation from `if-else` statements. The error is provided using an `Error` object. This ensures stack trace capture and proper handling.

Errors should only be used for invalid input or internal failures. For example, if you are executing a database query that returns no data, that is _not_ an error; it is just a fact about the database. If the database connection fails, that is an error.

The code for this example is in the [sum-valid.js][] file.

## Organising patterns into plugins

A Seneca instance is just a set of action patterns. You can organize action patterns using namespacing conventions, such as `role:math`. To help with logging and debugging, Seneca supports a minimalist notion of a plugin.

Likewise, a Seneca plugin is just a set of action patterns. A plugin can have a name, which is used to annotate logging entries. Plugins can be given a set of options to control their behavior. Plugins also provide a mechanism for executing initialization functions in the correct order. For example, you want your database connection to be established before you try to read data from the database.

A Seneca plugin is a function that has a single parameter `options`. You pass this plugin definition function to the `seneca.use` method. Here is the minimal Seneca plugin (it does nothing!):

``` js
function minimal_plugin(options) {
  console.log(options)
}

require('seneca')()
  .use(minimal_plugin, {foo: 'bar'})
```

The `seneca.use` method takes two parameters:

* `plugin`: plugin definition function or plugin name.
* `options`: options object for the plugin.

The sample code (in file [minimal-plugin.js][]) produces the following output:

``` js
$ node minimal-plugin.js
2016 ...    INFO    hello  ...
{ foo: 'bar' }
```

Seneca provides detailed logging information when it starts and also when running. Normally, the log level is set to `INFO` which means you don't see very much. To see all the logs, try this:

``` js
$ node minimal-plugin.js --seneca.log.all
... lots of log lines ...
```

You can narrow this down by [grepping][] the log output for log lines relevant to plugin definition:

``` js
$ node minimal-plugin.js --seneca.log.all | grep plugin | grep DEFINE
2016...    3qf7...    DEBUG    plugin    basic           DEFINE    {}
2016...    3qf7...    DEBUG    plugin    transport       DEFINE    {}
2016...    3qf7...    DEBUG    plugin    web             DEFINE    {}
2016...    3qf7...    DEBUG    plugin    mem-store       DEFINE    {}
2016...    3qf7...    DEBUG    plugin    minimal_plugin  DEFINE    {foo=bar}
```

You can see that Seneca loads four _built-in_ plugins by default: [basic][], [transport][],[web][] and [mem-store][]. These provide core functionalities for basic microservices. You can also see that your `minimal_plugin` is in the list as well, and also shown are the options you provided: `{foo=bar}`. The name `minimal_plugin` is obtained from the plugin definition function name, so you should always give your plugin definition function a name.

Let's give the plugin some action patterns. The `this` context variable of the plugin definition function is an instance of Seneca that you can use to do this. Here's a `math` plugin:

``` js
function math(options) {

  this.add('role:math,cmd:sum', function (msg, respond) {
    respond(null, { answer: msg.left + msg.right })
  })

  this.add('role:math,cmd:product', function (msg, respond) {
    respond(null, { answer: msg.left * msg.right })
  })

}

require('seneca')()
  .use(math)
  .act('role:math,cmd:sum,left:1,right:2', console.log)
```

Running this file [math-plugin.js][] generates the following output:

``` js
$ node math-plugin.js
2016...    INFO    hello   ...
null { answer: 3 }
```

Let's look at the logging output relevant to this plugin by grepping for the string "math":

``` js
$ node math-plugin.js --seneca.log.all | grep math
2016...    alqs...    DEBUG    delegate  {plugin$={name=math,tag=undefined},ungate$=true,fatal$=true}    29ny56
2016...    alqs...    DEBUG    register  init     math
2016...    alqs...    DEBUG    plugin    math     DEFINE    {}
2016...    alqs...    DEBUG    plugin    math     ADD    qlh13h47d0nu    cmd:sum,role:math    sum
2016...    alqs...    DEBUG    plugin    math     ADD    10lk4seu3aee    cmd:product,role:math    product
2016...    alqs...    DEBUG    plugin    math     options    set    {math={}}
2016...    alqs...    DEBUG    act                -    -    DEFAULT    {init=math,tag=}
2016...    alqs...    DEBUG    register  ready    math    {}
2016...    alqs...    DEBUG    register  install  math    {exports=[]:}
2016...    alqs...    DEBUG    act       math     -    IN    pg7er4ouia1p/u5hfgtpmkeoy    cmd:sum,role:math    {role=math,cmd=sum,left=1,right=2}    ENTRY    A;qlh13h47d0nu    -
2016...    alqs...    DEBUG    act       math     -    OUT    pg7er4ouia1p/u5hfgtpmkeoy    cmd:sum,role:math    {answer=3}    EXIT    A;qlh13h47d0nu    5
```

There is detailed logging information on the plugin definition and initialization, but you can mostly ignore this for now. The most interesting lines are the ones showing the addition of action patterns within the _math_ plugin, and then the execution of the `role:math,cmd:sum,left:1,right:2` action, showing the inbound and outbound messages:

``` js
...
20.. al.. DEBUG plugin math   ADD ql.. cmd:sum,role:math     sum
20.. al.. DEBUG plugin math   ADD 10.. cmd:product,role:math product
...
20.. al.. DEBUG act    math - IN  pg.. cmd:sum,role:math {role=math,cmd=sum,left=1,right=2} ENTRY A;ql.. -
20.. al.. DEBUG act    math - OUT pg.. cmd:sum,role:math {answer=3} EXIT A;ql.. 5
...
```

With Seneca, you build up your system by defining a set of patterns that correspond to messages. You organize these patterns into plugins to make logging and debugging easier. You then combine one or more plugins into microservices. You'll create a `math` microservice in the next section.

Plugins often need to do some initialization work, such as connecting to a database. You don't do this work in the body of the plugin definition function. The definition function is synchronous by design, because all it does is _define_ the plugin. In fact, you should not call `seneca.act` at all in the plugin definition - call `seneca.add` only.

To initialize a plugin, you add a special action pattern: `init:<plugin-name>`. This action pattern is called in sequence for each plugin. The init function _must_ call its `respond` callback without errors. If plugin initialization fails, then Seneca exits the Node.js process. You want your microservices to fail fast (and scream loudly) when there's a problem. All plugins must complete initialization before any actions are executed.

To demonstrate initialization, let's add simplistic custom logging to the _math_ plugin. When the plugin starts, it opens a log file and writes a log of all operations to the file. The file needs to open successfully and be writable. If this fails, the microservice should fail:

``` js
var fs = require('fs')

function math(options) {

  // the logging function, built by init
  var log

  // place all the patterns together
  // this make it easier to see them at a glance
  this.add('role:math,cmd:sum',     sum)
  this.add('role:math,cmd:product', product)

  // this is the special initialization pattern
  this.add('init:math', init)


  function init(msg, respond) {
    // log to a custom file
    fs.open(options.logfile, 'a', function (err, fd) {

      // cannot open for writing, so fail
      // this error is fatal to Seneca
      if (err) return respond(err)

      log = make_log(fd)
      respond()
    })
  }

  function sum(msg, respond) {
    var out = { answer: msg.left + msg.right }
    log('sum '+msg.left+'+'+msg.right+'='+out.answer+'\n')
    respond(null, out)
  }

  function product(msg, respond) {
    var out = { answer: msg.left * msg.right }
    log('product '+msg.left+'*'+msg.right+'='+out.answer+'\n')
    respond(null, out)
  }


  function make_log(fd) {
    return function (entry) {
      fs.write(fd, new Date().toISOString()+' '+entry, null, 'utf8', function (err) {
        if (err) return console.log(err)

        // ensure log entry is flushed
        fs.fsync(fd, function (err) {
          if (err) return console.log(err)
        })
      })
    }
  }
}

require('seneca')()
  .use(math, {logfile:'./math.log'})
  .act('role:math,cmd:sum,left:1,right:2', console.log)
```

In this plugin code, the patterns are organized at the top of the plugin so that they are easy to see. The action functions are defined below these patterns. You can also see how the options are used to provide the location for the custom log file (it should go without saying that this is not a way to do production logging!).

The initialization function `init` does some asynchronous file system work and so must complete before any actions can be performed. If it fails, the whole service fails to initialize. To see this in action, try changing the log file location to something invalid, such as `'/math.log'`.

This code is available in the [math-plugin-init.js][] file.

## Writing microservices

Let's turn the _math_ plugin into a real microservice. First, you need to get organized. The business logic of the _math_ plugin - that is, the functionality that it provides - is separate from whatever way it communicates with the outside world. Sometimes you might expose a web service; other times you might listen on a message bus.

It makes sense to put the business logic - that is, the plugin definition - in its own file. _Node.js_ modules are perfect for this:

``` js
module.exports = function math(options) {

  this.add('role:math,cmd:sum', function sum(msg, respond) {
    respond(null, { answer: msg.left + msg.right })
  })

  this.add('role:math,cmd:product', function product(msg, respond) {
    respond(null, { answer: msg.left * msg.right })
  })

  this.wrap('role:math', function (msg, respond) {
    msg.left  = Number(msg.left).valueOf()
    msg.right = Number(msg.right).valueOf()
    this.prior(msg, respond)
  })

}
```

This plugin is defined in the [math.js][] file. You export the plugin definition function and then call seneca.use with the name of the file. You can either [require][] it in or if you like to be terse, let Seneca make the `require` call:

``` js
// these are equivalent
require('seneca')()
  .use(require('./math.js'))
  .act('role:math,cmd:sum,left:1,right:2', console.log)

require('seneca')()
  .use('math') // finds ./math.js in local folder
  .act('role:math,cmd:sum,left:1,right:2', console.log)
```

The `seneca.wrap` method matches a set of patterns and overrides all of them with the same action extension function. This is the same as calling `seneca.add` manually for each one. It takes the following two parameters:

* `pin`: a pin is a pattern-matching pattern.
* `action`: action extension function.

A `pin` is a pattern that matches other patterns (it "pins" them). The pin `role:math` matches the patterns `role:math,cmd:sum` and `role:math,cmd:product` that are registered with Seneca.

In this case, you use `seneca.wrap` to make sure that the `left` and `right` properties are parsed as numeric values, even if they are provided as strings.

Sometimes it can be useful to see a visual tree of the patterns and any overrides in a Seneca instance. You can do this using the `--seneca.print.tree` command line option. The file [math-tree.js][] loads the _math_ plugin, but then does nothing:

```
require('seneca')()
  .use('math')
```

We're just using it to show the action tree:

``` js
$ node math-tree.js --seneca.print.tree
2016 ... INFO    hello    ...
Seneca action patterns for instance: 9vjqzroin2k4/1436455291148/78025/-
├─┬ cmd:sum
│ └─┬ role:math
│   └── # math, (s1a28),
│       # math, (sw9ew), sum
└─┬ cmd:product
  └─┬ role:math
    └── # math, (sxti2),
        # math, (b8gcw), product
```

Here, you can see the name/value pairs of the action patterns arranged in a tree structure as well as any overrides. Action functions are indicated by the format: # `plugin`, `(action-id)`, `function-name`.

Everything is still in the same process. Let's change that. First you need a microservice:

``` js
require('seneca')()
  .use('math')
  .listen()
```

Running this code ([math-service.js][]) starts a microservice process that listens on port 10101 for HTTP requests. This is _not_ a web server. In this case, HTTP is being used as the transport mechanism for messages.

You can try it out by sending a request to the microservice. Open the URL: `http://localhost:10101/act?role=math&cmd=sum&left=1&right=2` in a web browser or use curl on the command line:

``` bash
$ curl -d '{"role":"math","cmd":"sum","left":1,"right":2}' http://localhost:10101/act
```

What you get back is:

``` js
{"answer":3}
```

Next, you need a microservice client:

``` js
require('seneca')()
  .client()
  .act('role:math,cmd:sum,left:1,right:2',console.log)
```

Running this code ([math-client.js][]) starts a microservice client that sends this JSON message:

``` js
{ "role":"math", "cmd":"sum", "left":1, "right":2 }
```

to the math-service microservice above, which then responds with:

``` js
{ "answer":3 }
```

With Seneca, you create microservices by calling `seneca.listen` and you talk to the services using `seneca.client`. In the example, you are using the default settings for the client and server (communicate via HTTP over port 10101). Both `seneca.client` and `seneca.listen` accept the following parameters:

* `port`: optional integer; port number.
* `host`: optional string; host IP address.
* `spec`: optional object; full specification object.

**Note:** On windows machines, if no host is specified, the client will try to connect to `host` at `0.0.0.0`, which won't work. To get around this, set the `host` to be `localhost`.

As long as the client and listen parameters are the same, the two services can communicate. Here are some examples:

* `seneca.client(8080)` → `seneca.listen(8080)`
* `seneca.client(8080, '192.168.0.2')` → `seneca.listen(8080, '192.168.0.2')`
* `seneca.client({ port: 8080, host: '192.168.0.2' })` → `seneca.listen({ port: 8080, host: '192.168.0.2' })`

Seneca provides you with **transport independence** because your business logic does not need to know how messages are transported or which service will get them. This is specified in the service setup code or configuration. In this case, the code in the _math.js_ plugin _never_ changes.

The HTTP transport provides an easy way to integrate with Seneca microservices, but it does have all the overhead of HTTP. Another transport that you can use is direct TCP connections. Seneca provides both HTTP and TCP options via the built-in [transport][]. Let's move to TCP:

* `seneca.client({ type: 'tcp' })` → `seneca.listen({ type: 'tcp' })`

The default _client/listen_ configuration sends all messages that the client does not recognize over the listening server. Locally defined patterns are executed locally. It's usually preferable to specify exactly which patterns should be sent to which service. You can do this using a _pin_.

Let's put all this together into an example that sends `role:math` messages out over TCP on port 30303 (just an arbitrary port) and executes all other messages locally.

First, the listening service ([math-pin-service.js][]):

``` js
require('seneca')()

  .use('math')

  // listen for role:math messages
  // IMPORTANT: must match client
  .listen({ type: 'tcp', pin: 'role:math' })
```

Then, the client ([math-pin-client.js][]):

``` js
require('seneca')()

  // a local pattern
  .add('say:hello', function (msg, respond){ respond(null, {text: "Hi!"}) })

  // send any role:math patterns out over the network
  // IMPORTANT: must match listening service
  .client({ type: 'tcp', pin: 'role:math' })

  // executed remotely
  .act('role:math,cmd:sum,left:1,right:2',console.log)

  // executed locally
  .act('say:hello',console.log)
```

You can use filtered logging to trace the flow of messages. You can use the command line option `--seneca...` to control how Seneca runs, including the log output generated. Seneca logs have the following attributes (in order of importance):

* `date-time`: when the log entry occurred.
* `seneca-id`: identifier for the Seneca process.
* `level`: one of _DEBUG_, _INFO_, _WARN_, _ERROR_, _FATAL_.
* `type`: entry code, such as act, plugin, etc.
* `plugin`: plugin name (actions without a plugin have root$).
* `case`: entry case, such as _IN_, _OUT_, _ADD_, etc.
* `action-id/transaction-id`: tracing identifier, **stays the same over the network**.
* `pin`: the action pattern for this message.
* `message`: the inbound or outbound message (truncated if too long).

If you run the above processes with `--seneca.log.all` then you get all the logs. If you look at the entries, you can see Seneca booting up all the internal plugins:

``` js
$ node math-pin-service.js --seneca.log.all
... lots of logs ...

$ node math-pin-client.js --seneca.log.all
... lots of logs ...
```

It's hard to see the log entries that you care about; that is, the ones relevant to the _math_ plugin. To narrow down the output, try this:

``` js
$ node math-pin-service.js --seneca.log=plugin:math
20.. 85.. DEBUG    plugin    math  DEFINE    {}
20.. 85.. DEBUG    plugin    math  ADD    (f3ysr)    cmd:sum,role:math    sum
20.. 85.. DEBUG    plugin    math  ADD    (9mocb)    cmd:product,role:math    product
20.. 85.. DEBUG    plugin    math  ADD    (ydx70)    cmd:sum,role:math
20.. 85.. DEBUG    plugin    math  ADD    (ka7rj)    cmd:product,role:math
20.. 85.. DEBUG    plugin    math  options    set    {math:{}}
20.. 85.. DEBUG    act    math  IN    2682lsrziy1i/rst61586f7wl    cmd:sum,role:math    {role:math,cmd:sum,left:1,right:2}    ENTRY    (ydx70)    LISTEN    o2..    -
20.. 85.. DEBUG    act    math  IN    1k46jra7rhpd/rst61586f7wl    cmd:sum,role:math    {role:math,cmd:sum,left:1,right:2}    PRIOR;(ydx70)    (f3ysr)    -    -    -
20.. 85.. DEBUG    act    math  OUT    1k46jra7rhpd/rst61586f7wl    cmd:sum,role:math    {answer:3}    PRIOR;(ydx70)    (f3ysr)    -    -    0    -
20.. 85.. DEBUG    act    math  OUT    2682lsrziy1i/rst61586f7wl    cmd:sum,role:math    {answer:3}    EXI(ydx70)    LISTEN    o2..    1    -

$ node math-pin-client.js --seneca.log=pin:role:math
20.. o2.. DEBUG    act    remote$ IN    2682lsrziy1i/rst61586f7wl    role:math    {role:math,cmd:sum,left:1,right:2}    ENTRY    CLIENT    -    -    -
nu
20.. o2.. DEBUG    act    remote$ OUT    2682lsrziy1i/rst61586f7wl    role:math    {answer:3}    EXIT    CLIENT    -    85..    15    -
null { answer: 3 }
```

On the listening server, the setting `--seneca.log=plugin:math` narrows down the log to those entries where the `plugin` attribute is _math_. You can see the registration of the _math_ plugin and the addition of its action patterns. If you remember, you used `seneca.wrap` to override the basic actions with a string-to-integer conversion. That means that the `role:math` actions have two _ADD_ lines in the logs. You'll also notice that each action gets an indentifer of the form (abcde). You can use this to find out the exact action function that is executed when a message comes in.

When a message comes in, an `IN` log entry is created. When the action function provides a result, an 'OUT' log entry is created. In the listening server logs above, you can see this happening when the {role:math,cmd:sum,left:1,right:2} comes in over the network from the client.

Look carefully at the generated unique action identifers `2682lsrziy1i/rst61586f7wl` and `1k46jra7rhpd/rst61586f7wl`. These can be used to trace the flow of messages. The `IN` and `OUT` lines have the same action identifier. Furthermore, the second page, after the /, is a transaction identifer. All sub-actions triggered by an initial action have the same transaction identifier, in this case `rst61586f7wl`.

The action identifier persists over the network so that you can trace message flows when you have many microservices. Let's look at the client side. The setting `--seneca.log=pin:role:math` is another filter. This time it filters all log entries where the action pattern contains the specific pin, in this case `role:math`.

You can see that the action identifier `2682lsrziy1i/rst61586f7wl` originated on the client, and is passed over the listening server. You can also see that the client and listening server identifiers, `85..` and `o2..` (shortened for clarity) are given in the log entries so that you can tell which service sent which message.

Also seen in the output of the client is the console.log printing of the results — these are not part of the logs, just ordinary printed output.

There are many ways to configure your microservice communciation architecture. Take a look at the reference links at the end of this guide for more information.

## Web Server Integration

Seneca is not a web framework. But you still need to connect it up to your web service API. Here's the easiest way to do that.

The most important thing to remember is that you don't want to expose your internal action patterns to the outside world. That's not good security practice. Instead, define a set of API patterns, say with property `role:api`. Then you can hook them up to your internal microservices.

Let's look at a simple example using [Express][]. Here's the Express app ([app.js][]):

``` js
var seneca = require('seneca')()
      .use('api')
      .client({ type:'tcp', pin:'role:math' })

var app = require('express')()
      .use(require('body-parser').json())
      .use(seneca.export('web'))
      .listen(3000)
```

You create a seneca instance, load the _api_ plugin, and then use `seneca.client` to send any `role:math` actions out to an external service. Your Express app is the microservice client.

The integration between Seneca and Express happens in this line:

``` js
      .use(seneca.export('web'))
```

Seneca exports a middleware function that Express can use.

Here's the _api_ plugin ([api.js][]):

``` js
module.exports = function api(options) {

  var valid_ops = { sum:'sum', product:'product' }

  this.add('role:api,path:calculate', function (msg, respond) {
    this.act('role:math', {
      cmd:   valid_ops[msg.operation],
      left:  msg.left,
      right: msg.right,
    }, respond)
  })


  this.add('init:api', function (msg, respond) {
    this.act('role:web',{use:{
      prefix: '/api',
      pin:    'role:api,path:*',
      map: {
        calculate: { GET:true, suffix:'/:operation' }
      }
    }}, respond)
  })

}
```

This is a normal Seneca plugin. The `role:api,path:calculate` pattern will be exposed via URL endpoint. The inbound message should have the property `operation` that specifies the calculation to perform: `sum` or `product`. You can see that the code creates an explicit message for `role:math`, and makes a small attempt to sanitize the input. The `seneca.act` method allows you to build up the message both from a string (in this case `'role:math'`), and an object, merging the two together as a convenience.

**Never use external input to create an action string. Always explicitly create the internal message. This avoids injection attacks.**

The initialisation action makes a call to the pattern `role:web`, and defines the property `use`. This is a definition object that defines a route mapping from URLs to action patterns. It has the properties:

* `prefix`: the URL prefix
* `pin`: the set of patterns to map
* `map`: the list of pin wildcard property values to use as URL endpoints.

Your URL endpoint starts with `/api/....`

The pin is `role:api,path:*`. This means map any patterns that have _role = 'api'_, and where a _path_ property is defined. In this case, there is only one match: `role:api,path:calculate`.

The map has the property `calculate` corresponding to the value of the path property.

Your URL endpoint starts with `/api/calculate/....`

The calculate property has a subobject that indicates that the HTTP method is GET, and that the URL should have a parameterised suffix (these work just like Express parameters).

Your full URL endpoint is `/api/calculate/:operation`.

The remaining message properties are obtained from the URL query string, and from any JSON body submitted with the HTTP request. In this case, we are using GET, so there is no body.

You can re-use the microservice from the previous example. To run the app, start:

``` js
$ node math-pin-service.js --seneca.log=plugin:math
... logs as above ...

$ node app.js --seneca.log=plugin:web,plugin:api
... log entries for the web and api plugins ...
```

To exercise the app, load these URLs in a browser:

``` js
http://localhost:3000/api/calculate/sum?left=2&right=3 → {"answer":5}

http://localhost:3000/api/calculate/product?left=2&right=3 → {"answer":6}
```

If you look at the log output, you can see the corresponding action calls. You'll also see the line:

``` js
20.. ks.. DEBUG    plugin    web  ACT  f7../b8..  role:web  http  get  /api/calculate/:operation
```

Look for lines like this to see what URL endpoints you are building.

## Data Storage

You'll need to persist your data. Especially if you plan to build real-world systems! You can do anything you like inside Seneca actions, and use any kind of database layer. However, why not use the power of pattern matching and microservices to make your life easier?

The pattern matching approach also means you can postpone the debate about microservice data — do services "own" data, do they access a shared database, etc. The pattern matching approach means you can reconfigure your system any which way later on.

[seneca-entity][] provides a simple data abstraction layer ("ORM"), based on the following operations:

* **load**: load an entity by identifier
* **save**: create or update (if you provide an identifier) an entity
* **list**: list entities matching a simple query
* **remove**: delete an entity by identifier

The patterns are:

* **load**: `role:entity,cmd:load,name:<entity-name>`
* **save**: `role:entity,cmd:save,name:<entity-name>`
* **list**: `role:entity,cmd:list,name:<entity-name>`
* **remove**: `role:entity,cmd:remove,name:<entity-name>`

A plugin can provide access to a database (say [MySQL][]) by providing implementations of these patterns.

Microservice development is made much easier when data persistence is provided by the same mechanism as everything else: pattern-matched messages.

Using the data persistence patterns directly can become tedious, so [seneca-entity][] also provides a more familiar [ActiveRecord][]-style interface. To create a record object, you call the `seneca.make` method. The record object has methods `load$`, `save$`, `list$` and `remove$` (the trailing $ avoids clashes with data fields). The data fields are just the object properties.

To use [seneca-entity][] include the module in your package.json and add a seneca.use statement to require it into your seneca instance.

Let's create and save a simple data entity that stores "product" details:

``` js
var seneca = require('seneca')()
seneca.use('entity')

var product = seneca.make('product')
product.name = 'Apple'
product.price = 1.99

// sends role:entity,cmd:save,name:product messsage
product.save$( console.log )
```

Run the file <a href="">product.js</a> to test this. You'll see the output:

``` js
2016 ... INFO    hello    ...
null $-/-/product:{id=3i402d;name=Apple;price=1.99}
```

The response to the `role:entity,cmd:save message` is the record object, which prints itself as `$-/-/product:{id=3i402d;name=Apple;price=1.99}`. You can see that it auto-generated an identifier for you.

Seneca comes with a built-in data persistence plugin: [mem-store][]. This plugin just stores the data in-memory, and does not actually persist it anywhere. It's very useful for writing fast unit tests!

Because all data operations go via the same set of messages, you can very easily swap databases, at any time. No need to make database choice at the start of your project! Maybe use [MongoDB][] to begin with when your schemas are in development, and switch to [Postgres][] for the final few months before go-live and production.

Let's build a little shop, and integrate it into our existing microservice system. First, here's a simple shop plugin, with some messages for adding products, getting product details, and making a purchase ([shop.js][]):

``` js
module.exports = function( options ) {

  this.add( 'role:shop,get:product', function( msg, respond ) {
    this.make( 'product' ).load$( msg.id, respond )
  })

  this.add( 'role:shop,add:product', function( msg, respond ) {
    this.make( 'product' ).data$(msg.data).save$(respond)
  })

  this.add( 'role:shop,cmd:purchase', function( msg, respond ) {
    this.make( 'product' ).load$(msg.id, function( err, product ) {
      if( err ) return respond( err )

      this
        .make( 'purchase' )
        .data$({
          when:    Date.now(),
          product: product.id,
          name:    product.name,
          price:   product.price,
        })
        .save$( function( err, purchase ) {
          if( err ) return respond( err )

          this.act('role:shop,info:purchase',{purchase:purchase})
          respond(null,purchase)
        })
    })
  })

  this.add( 'role:shop,info:purchase', function( msg, respond ) {
    this.log.info('purchase',msg.purchase)
    respond()
  })
}
```

The `role:shop,get:product` pattern retrieves a product from the "database". You provide the identifier via the `id` property. Notice the `respond` callback is simply passed along to the `load$` method to use as its callback.

The `role:shop,add:product` pattern adds a new product to the "database". You provide the data fields via the `data` object property. The `data$` method is a shortcut for setting all the data fields from the provided object (in this case, `msg.data`). Again, respond is passed along.

The `role:shop,cmd:purchase` pattern creates a new row in the `purchase` table (a small part of what would happen in the real world when you hit the _Checkout_ button). The `product` identifier and details for that transaction are recorded (prices change!). The action function also emits a `role:shop,info:purchase` message, **but does not expect a response**. This is a [common microservice pattern][] — letting the world know that something has happened, but not caring who gets the message.

Finally, you provide a default implementation for the `role:shop,info:purchase`message. This is useful for debugging and unit testing. In the example code, you can see that it uses the `seneca.log.info` method to log the purchase event. The `seneca.log` object provides a method for each of the log levels: `debug, info, warn, error, fatal`. These methods also annotate your log entries with the name of your plugin.

Let's create simple unit test for this plugin. Writing unit tests for Seneca plugins is very easy — verify that inbound messages generate the right responses. Here's the unit test code ([shop-test.js][]):

``` js
var assert = require('assert')

var seneca = require('seneca')()
      .use('entity')
      .use('shop')

      // uncomment to send messages to the shop-stats service
      // .client({port:9003,pin:'role:shop,info:purchase'})

      .error( assert.fail )

add_product()

function add_product() {
  seneca.act(
    'role:shop,add:product,data:{name:Apple,price:1.99}',
    function( err, save_apple ) {

      this.act(
        'role:shop,get:product', {id:save_apple.id},
        function( err, load_apple ) {

          assert.equal( load_apple.name, save_apple.name )

          do_purchase( load_apple )
        })
    })
}

function do_purchase( apple ) {
  seneca.act(
    'role:shop,cmd:purchase',{id:apple.id},
    function( err, purchase) {
      assert.equal( purchase.product, apple.id )
    }
  )
}
```

This code uses the built-in Node.js [assert][] module. It loads the shop plugin, and then exercises the `role:shop` messages. If you run the test file, you'll see the output:

```
$ node shop-test.js
20.. h3.. INFO hello  ...
20.. h3.. INFO plugin shop ACT gj.. info:purchase,role:shop purchase {when:1436528824554,product:90vzcc,name:Apple,price:1.99,id:3i402d}
```

This output includes the log entry that you created with seneca.log.info in the implementation of the `role:shop,info:purchase` action.

Let's run a separate service to capture the purchase message events. For the sake of example we'll just count the number of purchases per product. Here's the [shop-stats.js][] microservice:

``` js
var stats = {}
require('seneca')()
  .add('role:shop,info:purchase',function( msg, respond ) {
    var product_name = msg.purchase.name
    stats[product_name] = stats[product_name] || 0
    stats[product_name]++
    console.log(stats)
    respond()
  })
  .listen({port:9003,pin:'role:shop,info:purchase'})
```

This service listens on port 9003, and prints out a product purchase statistics report every time a new purchase is made. Notice that you are providing an implementation of `role:shop,info:purchase`.

To see this in action, uncomment the line in [shop-test.js][]:

``` js
      // uncomment to send messages to the shop-stats service
      // .client({port:9003,pin:'role:shop,info:purchase'})
```

And then run both:

``` js
$ node shop-test.js
20.. j0.. INFO hello  ...
20.. j0.. INFO client {port:9003,pin:role:shop,info:purchase}

$ node shop-stats.js
20.. wb.. INFO hello  ...
20.. wb.. INFO listen {port:9003,pin:role:shop,info:purchase}
{ Apple: 1 }
```

## Bringing it Altogether

You're going to run four services. In the real world, you'd use something like <a href="https://www.docker.com">Docker</a> to keep yourself sane. For the purposes of this example, you'll run everything in the terminal as bare processes.

The services are:

* [shop-stats.js][]: collect shop statistics
* [shop-service.js][]: provide shop functionality
* [math-pin-service.js][]: provide math functionality (as above)
* [app-all.js][]: web server

The services `shop-stats` and `math-pin-service` are the same as before, so you can spin them up right away:

``` js
$ node math-pin-service.js --seneca.log.all

$ node shop-stats.js --seneca.log.all
```

In this example, we're using `--seneca.log.all` to log at the highest level of detail. There's a lot of output. Look out for the `act` lines, and the `IN` and `OUT` cases, and you can trace the message flows.

We need a [shop-service.js][]:

``` js
require( 'seneca' )()
  .use('entity')
  .use( 'shop' )
  .listen( { port:9002, pin:'role:shop' } )
  .client( { port:9003, pin:'role:shop,info:purchase' } )
```

This service listens for inbound `role:shop` messages, **but** sends any `role:shop,info:purchase` messages out onto the network. Seneca lets you mix and match `client` and `listen` configurations. **Remember, the client and listen pins must match**.

In this configuration, we've put the `shop-service` on local port 9002, and the `shop-stats` service on local port 9003. In production, you might use a [message bus][], or have [multiple clients][], or an [overlay network][], or even [service discovery][] to configure the port and host.

Start the `shop` service:

``` js
$ node shop-service.js --seneca.log.all
```

The shop functionality is exposed via the URL endpoints `/api/shop/get` and `/api/shop/purchase`. We need to add these to the _api_ plugin ([api-all.js]):

``` js
  ...

  this.add( 'role:api,path:shop', function( msg, respond ) {
    var shopmsg = { role:'shop', id:msg.pid }
    if( 'get'      == msg.operation ) shopmsg.get = 'product'
    if( 'purchase' == msg.operation ) shopmsg.cmd = 'purchase'

    this.act( shopmsg, respond )
  })

  this.add( 'init:api', function( msg, respond ) {

    ...

  this.act('role:web',{use:{
    prefix: '/api',
    pin:    'role:api,path:*',
    map: {
      shop: { GET:true, POST:true, suffix:'/:operation' },
    }
  }})

  respond()
})
```

Finally, we need to update the web server to send `role:shop` messages to the `shop-service` ([app-all.js][]):

``` js
var seneca = require( 'seneca' )()
      .use('entity')
      .use( 'api-all' )
      .client( { type:'tcp', pin:'role:math' } )
      .client( { port:9002,  pin:'role:shop' } )

var app = require( 'express' )()
      .use( require('body-parser').json() )
      .use( seneca.export( 'web' ) )
      .listen(3000)

// create a dummy product
seneca.act(
  'role:shop,add:product',{data:{name:'Apple',price:1.99}},
  console.log
)
```

We've also added a dummy product for testing. When you run the web service with:

``` js
$ node app-all.js --seneca.log.all
```

the last line of the output will be something like:

``` js
null $-/-/product:{id=mbm07t;name=Apple;price=1.99}
```

Copy the product identifier, in this case `mbm07t`. You can use this to exercise the system.

First, get the product details:

``` js
http://localhost:3000/api/shop/get?pid=mbm07t → {"name":"Apple","price":1.99,"id":"mbm07t"}
```

Then, make a purchase:

``` js
$ curl -d '{"pid":"mbm07t"}' -H "content-type:application/json" http://localhost:3000/api/shop/purchase
{"when":1436536799159,"product":"mbm07t","name":"Apple","price":1.99,"id":"ny09dx"}
```

Using `curl` on the command line, you can create a HTTP POST request to make a purchase. Note the need for the correct content type header: `application/json`.

Look at the logging output of all services. You'll be able to trace the action identifiers and transaction identifiers across all the services.

Don't forget that the _math_ service still works:

``` js
http://localhost:3000/api/calculate/sum?left=2&right=3 → {"answer":5}
```

**The math and shop services can be changed, updated, deployed, or even removed independently.**

Changes to one service do not affect the others. This is how microservices give you [continuous delivery][].

[getting-started-repo]: https://github.com/senecajs/getting-started
[seneca-in-practice]: https://github.com/senecajs/seneca-in-practice
[seneca-entity]: https://github.com/senecajs/seneca-entity
[sum.js]: https://github.com/senecajs/getting-started/blob/master/sum.js
[api-doc]: http://senecajs.org/api/
[Node.js]: https://nodejs.org/en/
[Error]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error
[sum-integer.js]: https://github.com/senecajs/getting-started/blob/master/sum-integer.js
[jsonic]: https://github.com/rjrodger/jsonic
[sum-reuse.js]: https://github.com/senecajs/getting-started/blob/master/sum-reuse.js
[pattern-wins.js]: https://github.com/senecajs/getting-started/blob/master/pattern-wins.js
[patrun module]: https://www.npmjs.com/package/patrun
[sum-valid.js]: https://github.com/senecajs/getting-started/blob/master/sum-valid.js
[minimal-plugin.js]: https://github.com/senecajs/getting-started/blob/master/minimal-plugin.js
[grepping]: http://www.cyberciti.biz/faq/howto-use-grep-command-in-linux-unix/
[basic]: https://www.npmjs.com/package/seneca-basic
[transport]: https://www.npmjs.com/package/seneca-transport
[web]: https://www.npmjs.com/package/seneca-web
[mem-store]: https://www.npmjs.com/package/seneca-mem-store
[math-plugin.js]: https://github.com/senecajs/getting-started/blob/master/math-plugin.js
[math-plugin-init.js]: https://github.com/senecajs/getting-started/blob/master/math-plugin-init.js
[math.js]: https://github.com/senecajs/getting-started/blob/master/math.js
[require]: https://nodejs.org/api/modules.html
[math-tree.js]: https://github.com/senecajs/getting-started/blob/master/math-tree.js
[math-client.js]: https://github.com/senecajs/getting-started/blob/master/math-client.js
[math-service.js]: https://github.com/senecajs/getting-started/blob/master/math-service.js
[math-pin-service.js]: https://github.com/senecajs/getting-started/blob/master/math-pin-service.js
[Express]: http://expressjs.com
[app.js]: https://github.com/senecajs/getting-started/blob/master/app.js
[api.js]: https://github.com/senecajs/getting-started/blob/master/api.js
[MySQL]: https://www.npmjs.com/package/seneca-mysql-store
[ActiveRecord]: https://en.wikipedia.org/wiki/Active_record_pattern
[mem-store]: https://www.npmjs.com/package/seneca-mem-store
[MongoDB]: https://www.npmjs.com/package/seneca-mongo-store
[Postgres]: https://www.npmjs.com/package/seneca-postgres-store
[common microservice pattern]: http://oredev.org/oredev2013/2013/wed-fri-conference/implementing-micro-service-architectures.html
[shop-test.js]: https://github.com/senecajs/getting-started/blob/master/shop-test.js
[assert]: https://nodejs.org/api/assert.html
[shop.js]: https://github.com/senecajs/getting-started/blob/master/shop.js
[shop-stats.js]: https://github.com/senecajs/getting-started/blob/master/shop-stats.js
[shop-service.js]: https://github.com/senecajs/getting-started/blob/master/shop-service.js
[math-pin-client.js]: https://github.com/senecajs/getting-started/blob/master/math-pin-client.js
[app-all.js]: https://github.com/senecajs/getting-started/blob/master/app-all.js
[message bus]: https://www.npmjs.com/package/seneca-amqp-transport
[multiple clients]: https://www.npmjs.com/package/seneca-redis-transport
[overlay network]: https://github.com/coreos/flannel
[service discovery]: https://www.consul.io
[continuous delivery]: https://www.thoughtworks.com/talks/software-development-21st-century-xconf-europe-2014
[api-all.js]: https://github.com/senecajs/getting-started/blob/master/api-all.js
