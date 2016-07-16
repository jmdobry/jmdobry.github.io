---
layout: post
title: "Let express.js and async.js make your life easier"
tags: javascript, NodeJS, express.js, async.js
---
Out of the box node.js provides an http module for sending and receiving HTTP
requests. It attempts to abstract away the more tedious parts of forming
requests and parsing responses. With the http module you can write scripts that
communicate with the world via HTTP or even write simple HTTP web servers. This
is great, but when it comes time to build a service api, node's http module
starts to feel like a single screwdriver when you need an entire toolbox.
[ExpressJS](http://expressjs.com/) comes into the picture as a minimal and
flexible web application framework for node.js. Express provides a set of tools
for easily interacting with HTTP requests and responses, in addition to bringing
middleware layers to your api. In addition to the fundamental features provided
by Express, the highly popular [async](https://github.com/caolan/async) utility
provides a robust tool for managing asynchronous control flow. Consider this
RESTful api endpoint created using Express:

```js
app.get('/posts/:id', function (req, res, next) {
  db.find(req.params.id, function (err, posts) {
    if (err) {
      // Pass the error on to be handled by some other middleware
      next(err);
    } else {
      res.send(200, posts);
    }
  });
});
```

Here, syntactic sugar lays out an endpoint that accepts GET requests. Express
automatically parses parameters within the request url, and exposes them in the
params object of the request.

Notice the line:

```js
if (err) {
  next(err);
}
```

This is where the code checks for any errors returned by the database call.
Instead of handling the error on the spot with something like:

```js
if (err) {
  // Handle the error on the spot
  res.send(500, 'There was an error!');
}
```

I pass the error down the middleware chain to be handled elsewhere. Requests
pass through middleware in the order that you tell Express to "use" them (see
code below). This is convenient when you want to handle errors from many places
with the same code. In order for this to work, previously in my application I
had told Express to add a handler to the middleware chain with something like
this:

```js
// This tells express to route ALL requests through this middleware
// This middleware ends up being a "catch all" error handler
app.use(function (err, req, res, next) {
  if (err.msg) {
    res.send(500, { error: err.msg });
  } else {
    res.send(500, { error: '500 - Internal Server Error' });
  }
});
```

So when the request (GET "/posts/13") comes from the client to Express, the
request is passed down the middleware chain until it is handled. Most middleware
don't fully handle the request, but augment it. For example, Expresses uses
middleware to parse the url parameters and attach the params object to the
request object, which is then passed along to be handled by the code written to
handle the "/posts/:id" route. If there's an error, the route can handle the
error, or pass it along to be handled by a middleware error handler like the one
above.

Middleware is also especially useful for requiring authentication in api
endpoints. For example, we will secure the "/posts/:id" route:

```js
function restrict(req, res, next) {
  if (req.session.user) {
    // User has active authenticated session, move along
    next();
  } else {
    // Unauthenticated!
    res.send(401);
  }
}
```

And now to tell Express to send the request through the "restrict" middleware
before letting the code for the "/posts/:id" route handle the request:

```js
app.get('/posts/:id', restrict, function (req, res, next) {
  db.find(req.params.id, function (err, posts) {
    if (err) {
      next(err);
    } else {
      res.send(200, posts);
    }
  });
});
```

Using middleware, it's possible to build a robust and highly functional service
api without duplicating code.

When handling a request, and especially when that involves doing file I/O or
talking to a database, it often leads what's called "callback hell", or deeply
nested callback functions. This makes the code hard to read, as it's hard to
follow the flow of the program. Here's an example of trying to create a new
user:

```js
function create(attrs, cb) {
  db.table('user').filter({email: attrs.email}).count().run(db.conn, function (err, count) {
    if (count > 0) {
      cb('Email already exists!');
    } else {
      db.table('user').filter({username: attrs.username}).count().run(db.conn, function (err, count) {
        if (count > 0) {
          cb('Username already exists!');
        } else {
          db.table('user').insert(attrs).run(db.conn, function (err, newUser) {
            if (err) {
              cb(err);
            } else {
              cb(null, newUser);
            }
          });
        }
      });
    }
  });
}
```

Imagine how painful it would be to read code nested eight or ten levels deep!
Thankfully, async can help us out. Here's the same example using async:

```js
function create(attrs, cb) {
  async.waterfall([
    // check for duplicate email
    function (next) {
      db.table('user').filter({email: attrs.email}).count().run(db.conn, next);
    },
    // check for duplicate username
    function (count, next) {
      if (count > 0) {
        cb('Email already exists!');
      } else {
        db.table('user').filter({username: attrs.username}).count().run(db.conn, next);
      }
    },
    // Insert the new user
    function (count, next) {
      if (count > 0) {
        cb('Username already exists!');
      } else {
        db.table('user').insert(attrs, { 'return_vals': true }).run(db.conn, next);
      }
    }
  ]);
}
```

Now each step is listed in its logical order, at the same level of nesting. This
particular function of async–waterfall–performs asynchronous operations one
after another, each waiting for the previous to complete before executing, and
each receiving the return arguments of the preceding function. Async
automatically checks for errors and returns them, eliminating the `if (err) cb(err);`
boilerplate code.

Using the combination of Express and async on one of my projects has been a
pleasant exercise that's resulted in a clear, concise api definition and easy to
follow code. I would definitely give async a try, as it can do _almost_
everything promises can do, without the performance overhead.
[async](https://github.com/caolan/async) has a myriad of functions you can check
out.
