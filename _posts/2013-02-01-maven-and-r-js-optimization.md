---
layout: post
title: "Maven and r.js optimization"
tags: maven, r.js
---
Using Maven to manage a large project has it's advantages, until you need to do
something that doesn't fit into the Maven's defined context. Tools based on
composability provide components that can be combined in various ways to
accomplish needed tasks. Maven—a contextual tool—provides a lot of functionality
out of the box, but ultimately requires users to conform to Maven's way of doing
things, not allowing the user to simple string together commands to perform
custom tasks.

I personally am a big fan of using [Grunt.js](http://gruntjs.com/)—a composable
tool—for performing tasks on my JavaScript (like r.js optimization). However,
Grunt introduces a dependency on Node.js—a JavaScript run-time. Maven's design
does not sit well with running external scripts during a Maven build. How can
these scripts communicate back to Maven and report on their success, failure, or
something in between? They can't. Maven watches for a successful return code,
but that's it. External scripts are used because Maven didn't provide the tools
to perform the tasks, so 3rd-party tools were brought into the picture. But
because the extra tools weren't grown in Maven's context, they're on their own.

I messed around with having Maven run external node.js scripts for a while, but
it was ugly. I decided to give mcheely's [requirejs-maven-plugin](https://github.com/mcheely/requirejs-maven-plugin)
a try. His plugin takes the r.js optimizer and shoves it into Maven's context.
And you know what? It works pretty well.

Configuration can be as simple as:
```xml
<plugin>
  <groupId>com.github.mcheely</groupId>
  <artifactId>requirejs-maven-plugin</artifactId>
  <version>1.0.4</version>
  <executions>
    <execution>
      <phase>process-classes</phase>
      <goals>
        <goal>optimize</goal>
      </goals>
    </execution>
  </executions>
  <configuration>
    <configFile>${basedir}/src/main/webapp/resources/js/app.build.js</configFile>
    <!-- whether or not to process config with maven filters -->
    <filterConfig>false</filterConfig>
    <!-- Skip requirejs optimization if true -->
    <skip>false</skip>
  </configuration>
</plugin>
```

By using this plugin (and a few others like the [minify-maven-plugin](http://samaxes.github.com/minify-maven-plugin/ "minify-maven-plugin"))
here at work we've successfully eliminated our dependency on Node.js for builds,
which makes development environments easier to setup for everybody and means
less things for Java developers to have to learn. I have nothing against Node.js
here—I really like it, but the existence of plugins like this make life with
Maven (and other devs) a little easier.