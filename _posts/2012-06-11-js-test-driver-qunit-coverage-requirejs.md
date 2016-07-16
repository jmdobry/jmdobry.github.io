---
layout: post
title: "JsTestDriver + QUnit + coverage + RequireJS"
tags: require.js, testing, jstestdriver, javascript
---
#### Update:
Check out my updated post on [using jsTestDriver](/backbone-testing/ "Backbone-testing using jsTestDriver and Jasmine").

Thanks to new frameworks and tools like [Backbone.js](http://backbonejs.org/ "Backbone.js") and [Require.js](http://requirejs.org/ "Require.js") (mentioned here because they're my favorite), many developers are beginning to write web applications in JavaScript the way they have been writing them in Java. Obviously JavaScript was not originally purposed to be used this way, so the transition is difficult. Some of the more difficult transition areas I have found are those of automated Unit Testing and Continuous Integration. Testing frameworks like QUnit and Jasmine exist to help, but they don't change the fact that JavaScript isn't made up of distinct units, so to speak, unless of course you use something like Backbonejs and Requirejs. Writing Backbone modules AMD style makes it almost too easy to write unit tests, so I won't go into that. I chose QUnit here because I figured it out faster than Jasmine, and then I threw in Sinon.js to make my QUnit framework as feature rich as Jasmine. The Sinon-QUnit adapter integrates Sinon into QUnit for me.  Here are some links: [qunit.js](https://github.com/jquery/qunit "qunit.js"), [sinon.js](http://sinonjs.org/ "sinon.js"), [sinon-qunit.js](http://sinonjs.org/qunit/ "sinon-qunit.js").

What really interested me was being able to automate my unit tests and run them cross-browser. A single Headless WebKit won't cut it. I settled on [JSTestDriver](http://code.google.com/p/js-test-driver/ "JSTestDriver") because it does just what I need and is under active development, albeit slowly. It works out of the box by itself (see [Getting Started](http://code.google.com/p/js-test-driver/wiki/GettingStarted "Getting Started with JSTestDriver")) and even has a QUnit to JSTestDriver adapter . However, the adapter on the JSTestDriver website is way out of date, so I've provided links to the most recent and complete versions I could find. [equiv.js](https://github.com/exnor/QUnit-to-JsTestDriver-adapter/blob/master/equiv.js "equiv.js"), [QUnitAdapter.js](https://github.com/exnor/QUnit-to-JsTestDriver-adapter/blob/master/QUnitAdapter.js "QUnitAdapter.js"). This adapter makes it so you can write all of your test cases in QUnit, and JSTestDriver will run them all.

My last requirement was [code coverage](http://code.google.com/p/js-test-driver/downloads/list "jsTestDriver code coverage plugin"). We use Sonar with our Java, so if JavaScript wants to play with the big boys he's going to have to act like one. The code coverage plugin for JSTestDriver makes this happen. The coverage results produced by JSTestDriver [can be used](http://docs.codehaus.org/display/SONAR/JavaScript+Plugin "Sonar JavaScript plugin") by Sonar nicely. While searching forums it seems that many have found it difficult to make JSTestDriver and Requirejs to play nicely, mainly because both are trying to handle the loading of files. It seems that the developers are working on making JSTestDriver AMD compatible, but for now I've found a temporary solution. One downside I've discovered is that your source files under test _cannot_ be anonymous modules. They have to have names (the name has to be exactly what you would use to reference the module normally) like so:

```js
define('models/user', [...], function(...) {
  return Backbone.Model.extend() {
     //...
  };
});
```

And each test file needs to be in this example format: I explain why below.

```js
require([
  // Load libs and src files under test.
  'util/logging',
  'models/user',
  'collections/users'
],
function (Log, User, Users) {
  module("Example Test Suite", {
    setup: function () {
      // setup stuff
    },
    teardown: function () {
      // teardown stuff
    }
  });

  test("Example test.", function () {
    equal(1, 1, "Passes with 1 == 1");
  });

  test("Example test 2", function () {
    strictEqual(1, 1, "Passes with 1 === 1");
  });
});
```

Normally I would write a 'testrunner' file that would load my test files as dependencies, and my test files would be written with `define([...], function( ...)};` but this doesn't play with JsTestDriver, so each test file is standalone, and I let JsTestDriver load them itself.

Example jsTestDriver.conf:

```
server: http://localhost:42442
# All urls below will be relative to this path.
basepath: C:/path/to/root/of/project/

load:

# Load test libraries here. I listed them so you could see in what order I load them.
- test/lib/qunit.js
- test/lib/sinon.js
- test/lib/sinon-qunit.js
- test/lib/equiv.js
- test/lib/QUnitAdapter.js

# Make sure you load Requirejs before you load any file that starts with "define" or "require".
- src/lib/require.js

# This config file is for require.js. This is different than the one you use in your production code
# because we need to add a BaseUrl that takes into account the directory to which js-test-driver
# uploads files to the server. See below for config.js.
- test/lib/config.js

# Load the remainder of your src libraries here. Backbone, Underscore, Text, JQuery, Handlebars, etc.
# Anything you normally load in production code.
- src/lib/*.js

# Load src files under test here.
- src/models/*.js
- src/collections/*.js
- src/views/*.js

# And finally we load all of our tests.
- test/*.js

# Send static files to the server.
serve:
- src/css/*.css
- src/templates/*.html

plugin:
- name: "coverage"
jar: "path/to/coverage/plugin/coverage-1.3.4.b.jar"
module: "com.google.jstestdriver.coverage.CoverageModule"

timeout: 90
```

In this config.js I add a baseUrl to allow the src files under test to load files relative to their new location, i.e., '/test/original/path/'. The paths will be the same as in your production code's requirejs config file.

```js
require.config({
  baseUrl: '/test/src/',
  paths: {
    jquery : 'lib/jquery/jquery.min',
    underscore : 'lib/underscore/underscore',
    backbone : 'lib/backbone/backbone',
    text : 'lib/require/text',
    handlebars: 'lib/handlebars/handlebars'
  }
});
```

It doesn't really matter where you place your JsTestDriver-1.3.4.b.jar relative to jsTestDriver.conf in your project, because when you run the jar you can just tell it where the config file is. Here are the two commands I use:

Start the server and capture browsers:

```
C:Work/exampleProject>java -jar JsTestDriver-1.3.4.b.jar --port 42442 --config path/to/jsTestDriver.conf --browser yourpathtofirefox,yourpathtochrome,yourpathtoIE
```

Run the tests:

```
C:Work/exampleProject>java -jar JsTestDriver-1.3.4.b.jar --config path/to/jsTestDriver.conf --runnerMode PROFILE --reset --dryRunFor all --tests all
```

With sample output:

```
C:Work/exampleProject>java -jar JsTestDriver-1.3.4.b.jar --config config/test/resources/jsTestDriver.conf --runnerMode PROFILE --reset --dryRunFor all --captureConsole --tests all
setting runnermode PROFILE
Microsoft Internet Explorer: Reset
Firefox: Reset
Chrome: Reset
Firefox 13.0.1: 5 tests [
  User Test Suite 1 (/test/test/userTest.js)
    test Correct RESTful service endpoint test.
    test Empty Test.,
  Users Test Suite 1 (/test/test/usersTest.js)
    test Correct RESTful service endpoint test.,
  Users Test Suite 2 (/test/test/usersTest.js)
    test Size Test
]
Microsoft Internet Explorer 9.0: 5 tests [
  User Test Suite 1 (/test/test/qunit/tests/models/userTest.js)
    test Correct RESTful service endpoint test.
    test Empty Test.,
  Users Test Suite 1 (/test/test/usersTest.js)
    test Correct RESTful service endpoint test.,
  Users Test Suite 2 (/test/test/usersTest.js)
    test Size Test
]
Chrome 19.0.1084.56: 5 tests [
    User Test Suite 1 (/test/test/userTest.js)
  test Correct RESTful service endpoint test.
    test Empty Test.,
  Users Test Suite 1 (/test/test/usersTest.js)
    test Correct RESTful service endpoint test.,
  Users Test Suite 2 (/test/test/usersTest.js)
    test Size Test
]
..................
Total 12 tests (Passed: 12; Fails: 0; Errors: 0) (20.00 ms)
Firefox 13.0.1 Windows: Run 4 tests (Passed: 4; Fails: 0; Errors 0) (20.00 ms)
Microsoft Internet Explorer 9.0 Windows: Run 4 tests (Passed: 4; Fails: 0; Errors 0) (6.00 ms)
Chrome 19.0.1084.56 Windows: Run 4 tests (Passed: 4; Fails: 0; Errors 0) (9.00 ms)
C:WorkexampleProjecttestlibqunit.js: 11.823362% covered
C:WorkexampleProjecttestlibsinon.js: 21.598001% covered
C:WorkexampleProjecttestlibsinon-qunit.js: 50.0% covered
C:WorkexampleProjecttestlibequiv.js: 12.987013% covered
C:WorkexampleProjecttestlibQUnitAdapter.js: 44.827587% covered
C:WorkexampleProjectsrclibjqueryjquery.min.js: 33.333336% covered
C:WorkexampleProjectsrclibhandlebarshandlebars.js: 10.059172% covered
C:WorkexampleProjectsrclibunderscoreunderscore.js: 31.275719% covered
C:WorkexampleProjectsrclibbackbonebackbone.js: 51.468052% covered
C:WorkexampleProjectsrclibrequirerequire.js: 55.172413% covered
C:WorkexampleProjecttestlibconfig.js: 100.0% covered
C:WorkexampleProjectsrclibrequiretext.js: 0.952381% covered
C:WorkexampleProjectsrcliblogging.js: 36.363636% covered
C:WorkexampleProjectsrcmodelsuser.js: 68.75% covered
C:WorkexampleProjectsrccollectionsusers.js: 100.0% covered
C:WorkexampleProjecttestuserTest.js: 100.0% covered
C:WorkexampleProjecttestusersTest.js: 100.0% covered
INFO: Jun 28, 2012 11:07:57 AM com.google.jstestdriver.ActionRunner runActions

C:Work/exampleProject>

```

Unfortunately the coverage plugin for jsTestDriver does not have a way to exclude files. This is a needed feature because we don't want skewed metrics or to instrument files for which we will never write unit tests (JQuery, etc.). I am satisfied with this solution for now, but the configuration overhead is a pain, and I am in need of some features. Otherwise it's pretty awesome!

p.s. Check out the jstd-maven-plugin for Continuous Integration.

## UPDATE

Thanks to some new developments it is now possible to exclude code from code coverage, and RequireJS can be used without the need to "name" your modules. JsTestDriver just became a much more viable solution for me. Check out [this thread](http://code.google.com/p/js-test-driver/issues/detail?id=309&q=serve) for excluding files, and pay attention to the link to [this repo](https://github.com/blittle/jstesting). Bret Little has a working demo. Though it uses Jasmine as the testing framework, it demonstrates how to exclude code from coverage analysis. I would grab the JsTestDriver.jar and coverage.jar found in the repo. It seems they have a patch that allows the exclusion to work.

As for not needing to name RequireJS modules anymore, they just need to use a "require" call instead of a "define" call. Oh, and load them in the "serve" section instead of the "load" section. That way RequireJS will be free to load them without JsTestDriver getting in the way.
