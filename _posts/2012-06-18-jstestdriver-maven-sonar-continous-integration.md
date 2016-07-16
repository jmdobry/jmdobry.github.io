---
layout: post
title: "JsTestDriver + Maven + Sonar - Continuous Integration"
tags: testing, jstestdriver, javascript, maven
---
#### Update:
Check out my updated post on [using jsTestDriver](/backbone-testing/ "Backbone-testing using jsTestDriver and Jasmine").

As a follow up to my post [js-test-driver-qunit-coverage-requirejs](http://pseudobry.com/js-test-driver-qunit-coverage-requirejs/ "js-test-driver-qunit-coverage-requirej") I describe here my efforts to add JsTestDriver to our Continuous Integration process. This post is really a continuation of the aforementioned article, so feel free to refer to it before diving into this one.

To start off I downloaded the [jstd-maven-plugin](http://code.google.com/p/jstd-maven-plugin/ "jstd-maven-plugin") and followed the [Getting Started](http://code.google.com/p/jstd-maven-plugin/wiki/GettingStarted "Getting Started") instructions. I added the dependency, repository, and plugin entries, with my plugin entry being configured as follows:

```xml
<plugin>
  <!-- Execute test driver jar to run tests against VM instances running particular browsers -->
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
        <!-- Build Test Server -->
        <server>http://url.of.your.server</server>
        <config>your/path/to/jsTestDriver.conf</config>
        <runnerMode>DEBUG</runnerMode>
        <verbose>true</verbose>
        <jar>your/path/to/JsTestDriver-1.3.4.b.jar</jar>
        <reset>true</reset>
        <dryRunFor>all</dryRunFor>
        <testOutput>your/path/to/target/jstestdriver</testOutput>
      </configuration>
    </execution>
  </executions>
</plugin>
```

An explanation of what each configuration option does can be found [here](http://code.google.com/p/js-test-driver/wiki/CommandLineFlags). Although the previously mentioned link says nothing about the "jar" property, I found it necessary to add it to make the plugin work. I set up a couple VMs that have instances of JsTestDriver running, and a couple more with different browsers captured by those JsTestDriver servers. Right now I'm testing on IE7, IE8, IE9, Firefox, and Chrome. Opera seems to crash JsTestDriver so I'm avoiding it for now. When left open too long IE tends to shrivel and die, so rebooting every day keeps them fresh. I configured the VMs to reboot every morning, and wrote simple scripts the start the JsTestDriver servers and open browsers to the capture url, e.g.
`http://10.26.57.14:42442/capture`.

## Sonar Integration

Sonar becomes useful in this situation when it has the [Sonar-Javascript-Plugin](http://docs.codehaus.org/display/SONAR/JavaScript+Plugin "Sonar-Javascript-Plugin") installed. After installing it all that is left to do is to tell Sonar where to find the coverage and test results that JsTestDriver produces. This is done by adding the following properties to your project's pom.xml:

```xml
<properties>
  ...
  <sonar.dynamicAnalysis>reuseReports</sonar.dynamicAnalysis>
  <sonar.javascript.jstestdriver.reportsfolder>path/to/your/target/jstestdriver</sonar.javascript.jstestdriver.reportsfolder>
  ...
</properties>
```

If your project has only JavaScript as its language then you can also add the

```xml
<sonar.language>js</sonar.language>
```

 property and then run Sonar as normal, otherwise a little more configuration is necessary. Our small project uses Java _and_ JavaScript, and since Sonar can only do one language per analysis, I had to find a workaround. A really cool feature of Sonar is branching. When running Sonar you can pass a parameter
`-Dsonar.branch=nameofyourbranch`

and when Sonar analyzes your project, the results will appear in your Sonar project list as "nameofyourproject-nameofyourbranch". Our solution is to have two branches of our one project. When Sonar runs it needs to know where your "src" and "test" directories are. By default our pom.xml is setup to run Java tests and run the Sonar analysis on Java. Our project's pom.xml has the following:

```xml
<properties>
  ...
  <sonar.branch>java</sonar.branch>
  <sonar.language>java</sonar.language>
  <sourceDir>src/main/java</sourceDir>
  <testSourceDir>src/test/java</testSourceDir>
  ...
</properties>
```

and

```xml
<build>
  <sourceDirectory>${sourceDir}</sourceDirectory>
  <testSourceDirectory>${testSourceDir}</testSourceDirectory>
  <plugins>
      ...
  </plugins>
</build>
```

In order run our Sonar analysis on our JavaScript we have the following profile:

```xml
<profile>
  <id>sonarjs</id>
  <properties>
    <sonar.branch>js</sonar.branch>
    <sonar.language>js</sonar.language>
    <sourceDir>src/path/to/javascript/source/files</sourceDir>
    <testSourceDir>src/path/to/javascript/test/files</testSourceDir>
  </properties>
</profile>
```

The Sonar JavaScript plugin comes with lots of configuration of its Linting tool, and picks up our JsTestDriver coverage results nicely.