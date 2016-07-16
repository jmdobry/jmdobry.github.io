---
layout: post
title: "Testing Backbone.js"
tags: technology, javascript, testing, backbone.js
---
## Introduction

Getting RequireJs, JsTestDriver, Backbone, Jasmine, and Maven to play nice is
like trying to get Romney and Obama to play nice at a debate. They don't seem to
like each other. Here I discuss my experience getting them (not the presidential
candidates) to work together.

Keep in mind that my example app is not an example of best coding practices, but
simply shows how to combine Backbone, RequireJS, and Jasmine with the likes of
JsTestDriver and Maven.

## Jasmine

A popular BDD-style JavaScript testing framework. Jasmine usually runs by
opening an html page in a browser, which executes the Jasmine script. Here we
are testing a Backbone application that uses RequireJS as its script loader. See
SpecRunner.html in the example code for more information.

## JsTestDriver

JsTestDriver is a test runner for in-browser JavaScript tests. [Several months ago](/js-test-driver-qunit-coverage-requirejs/) I [attempted to use it](/jstestdriver-maven-sonar-continuous-integration-2/), but gave up because the
stability was pitiful, and it was missing some important features. It seems to
have since stabilized. Now I am using JsTestDriver to run my tests in several
browsers, and to calculate the code coverage of my projects. Previously I chose
not to use JsTestDriver because it wasn't capable of excluding code from code
coverage. In addition, JsTestDriver required me to "name" my RequireJs modules,
or else it would throw an error.

I am now using a patched version of JsTestDriver that solves these problems. The
patched version resulted from [this thread](https://code.google.com/p/js-test-driver/issues/detail?id=309&amp;q=serve "JsTestDriver ") on JsTestDriver's site. The patched
jars can be found in my example code.

When JsTestDriver loads JavaScript from the 'load' or 'test' section it executes
the JavaScript on load. The patched jar allows us to load our application source
files(anonymous RequireJS modules) from the 'serve' section. This means that
when JsTestDriver executes require.js, require.js will be free to load the files
free from interference from JsTestDriver.

```
server: http://localhost:9876

load:
  - assets/js/libs/jquery.js
  - test/jasmine/vendor/jasmine.js
  - test/jasmine/vendor/JasmineAdapter.js
  - test/jasmine/vendor/sinon-1.3.1.js
  - assets/js/libs/require.js
  - test/jasmine/vendor/jasmine-sinon.js
  - test/jasmine/vendor/jasmine-jquery.js
  - test/jasmine/vendor/jasmine-ajax.js
  - test/jasmine/config/jsTestDriver_requirejsConfig_jasmine.js

test:
  - test/jasmine/spec/*.js

serve:
  - assets/js/libs/*.js
  - assets/js/plugins/*.js
  - app/templates/*.html
  - app/data/*.json
  - app/collections/*.js
  - app/models/*.js
  - app/views/*.js

plugin:
 - name: "coverage"
   jar: "test/jsTestDriver/coverage-patched-1.3.4.b.jar"
   module: "com.google.jstestdriver.coverage.CoverageModule"
   args: "includesRegex:.*?app,excludesRegex:.*?config|.*?assets|.*?vendor|.*?spec|.*?templates"
```

## Maven and jsTestDriver-maven-plugin

Apache Maven is a software project management and comprehension tool. If you
have a Backbone app inside your Java application, or if you simply want to use
Maven to manage your JavaScript project, then it is handy to be able to run your
JavaScript tests during your Maven build. The jsTestDriver-maven-plugin
activates jsTestDriver during the build.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>amd-testing.jasmine</groupId>
  <artifactId>maven-jsTestDriver-jasmine-backbone</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>maven-jsTestDriver-jasmine-backbone</name>
  <url>http://maven.apache.org</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <jscoverage.skip>false</jscoverage.skip>
    <sonar.dynamicAnalysis>reuseReports</sonar.dynamicAnalysis>
    <sonar.javascript.jstestdriver.reportsfolder>target/jsTestDriver/</sonar.javascript.jstestdriver.reportsfolder>
    <sourceDir>src/main/js</sourceDir>
    <testSourceDir>src/test/js/jasmine/spec</testSourceDir>
    <sonar.exclusions>config.js,main.js</sonar.exclusions>
    <sonar.language>js</sonar.language>
    <patch.skip>true</patch.skip>
  </properties>

  <dependencies>
    <dependency>
      <groupId>com.googlecode.jstd-maven-plugin</groupId>
      <artifactId>jstd-maven-plugin</artifactId>
      <version>1.3.2.5</version>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>com.googlecode.jstd-maven-plugin</groupId>
        <artifactId>jstd-maven-plugin</artifactId>
        <version>1.3.2.5</version>
        <executions>
          <execution>
            <id>run-tests</id>
            <phase>test</phase>
            <goals>
              <goal>test</goal>
            </goals>
            <configuration>
              <jar>${basedir}/src/test/jsTestDriver/jsTestDriver-patched-1.3.4.b.jar</jar>
              <config>${basedir}/src/test/js/jasmine/config/jsTestDriver_maven_jasmine.conf</config>
              <reset>true</reset>
              <tests>all</tests>
              <testOutput>${basedir}/target/jstestdriver</testOutput>
            </configuration>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
</project>
```

## Conclusion

I now have a stable solution for running my in-browser JavaScript unit tests. I
have a number of browsers running on VMs, all captured by a JsTestDriver server.
The test and code coverage results are readable by Sonar, if the pom.xml has the
right configuration properties. (See example above).

This might be a terse explanation, but I hope my example code can explain itself.