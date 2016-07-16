---
layout: post
title: "Jasmine + Maven + Requirejs + Coverage"
tags: jasmine.js, require.js, testing, jstestdriver, javascript, maven
---
In earlier posts I succeeded in integrating JsTestDriver, Maven, and Sonar, with tests written with QUnit and Sinon, all bundled up with Requirejs. The solution worked great except for one thing--JsTestDriver just dies sometimes. Frequently during use captured browsers would lock up, or fail to be captured, or crash during tests. JsTestDriver's instability forced me to look for another solution. And found one I have.

I'd heard a lot about Jasmine, but I wasn't sure what the big deal was. Coming from QUnit I was caught off guard by Jasmine's style. I don't know what it was, maybe it was the exotic syntax highlighting or the funky word use, but the first time I read through Jasmine's site I thought _this is weird_. I was used to QUnit's "Module. Test. Test. Test." So when I saw Jasmine's "Describe. It. It. It." I wondered what is going on. It wasn't until my problems with JsTestDriver that I come back to Jasmine. A good second look made me of the opinion that Jasmine's word choice is actually rather intuitive. But this article isn't about Jasmine, for I didn't switch to Jasmine for Jasmine's sake. What I really needed was reliable automation, and since I had discovered a sweet Jasmine-Maven-Plugin with awesome documentation, I needed to learn Jasmine to use it. So I did.

## jasmine-maven-plugin

Now for my experience with the [Jasmine-Maven-Plugin](http://searls.github.com/jasmine-maven-plugin/ "jasmine-maven-plugin"). Pretty smooth. I encountered a few bumps while trying to get it to play nice with Require.js, but with comes with some Require.js-specific stuff to make it work. Here's my pom.xml setup for the plugin:

```xml
<plugin>
  <groupId>com.github.searls</groupId>
  <artifactId>jasmine-maven-plugin</artifactId>
  <version>1.2.0.0</version>
  <extensions>true</extensions>
  <executions>
    <execution>
      <id>FIREFOX_3</id>
      <phase>test</phase>
      <configuration>
        <browserVersion>FIREFOX_3</browserVersion>
        <junitXmlReportFileName>TEST-FIREFOX_3-jasmine.xml</junitXmlReportFileName>
        <manualSpecRunnerHtmlFileName>FIREFOX_3-ManualSpecRunner.html</manualSpecRunnerHtmlFileName>
        <specRunnerHtmlFileName>FIREFOX_3-SpecRunner.html</specRunnerHtmlFileName>
      </configuration>
      <goals>
        <goal>test</goal>
      </goals>
    </execution>
  </executions>
  <configuration>
    <jsSrcDir>src/main/webapp/resources</jsSrcDir>
    <jsTestSrcDir>src/main/webapp/resources/test/jasmine/spec</jsTestSrcDir>
    <sourceExcludes>
      <exclude>**/node_modules/**/*.*</exclude>
    </sourceExcludes>
    <specRunnerTemplate>REQUIRE_JS</specRunnerTemplate>
    <scriptLoaderPath>assets/js/lib/require/require.js</scriptLoaderPath>
    <serverPort>8888</serverPort>
    <format>progress</format>
  </configuration>
</plugin>
```

I have only one execution shown in the code, but you can add more executions that run HTMLUnit for IE 6, or IE 7, etc. JsTestDriver required me to name my require.js modules, but the jasmine-maven-plugin does not, thankfully. Sonar can easily read the test output as long as the .xml file begins with "TEST".

## Code Coverage

Here's another handy plugin I found called the [saga-maven-plugin](http://timurstrekalov.github.com/saga/ "saga-maven-plugin") that generates a code coverage report for my JavaScript. The plugin generates an HTML report viewable in the browser, and a .dat file readable by Sonar. I had a little hiccup with Sonar's JavaScript/JsTestDriver plugin in that it won't read the .dat file unless it's named jsTestDriver-coverage.conf. My maven build runs a script that renames the total-coverage.dat produced by the saga-maven-plugin to jsTestDriver-coverage.conf so Sonar can read it. I tried the recommended solution on Sonar's website, but it didn't work. This plugin is super easy to setup.

```xml
<plugin>
  <groupId>com.github.timurstrekalov</groupId>
  <artifactId>saga-maven-plugin</artifactId>
  <version>1.1.0</version>
  <executions>
    <execution>
      <phase>verify</phase>
      <goals>
        <goal>coverage</goal>
      </goals>
    </execution>
  </executions>
  <configuration>
    <baseDir>src/main/webapp/resources/test/jasmine</baseDir>
    <includes>
      SpecRunner.html
    </includes>
    <outputDir>${project.build.directory}/jasmine</outputDir>
    <noInstrumentPatterns>
      <pattern>b.+/test/.+.jsb</pattern>
      <pattern>b.+/assets/.+.jsb</pattern>
    </noInstrumentPatterns>
    <outputStrategy>TOTAL</outputStrategy>
  </configuration>
</plugin>
```

The plugin is capable of generating total coverage reports or per-test reports. However, it currently interprets my many test modules loaded from one SpecRunner.html as one test. Other than that I am satisfied. In the future I would like to see an all-in-one solution that runs my tests and instruments my source code at the same time. Right now my tests are run twice, which takes more time.

There is one more limitation that makes this solution less-than-perfect for my needs. HTMLUnit. The lack of a real DOM makes it hard (in fact it causes the plugin to crash if I try) to test DOM-related code. I've achieved 70% code coverage, but it's near impossible to test some of my remaining code because of this problem. On the bright side, this solution is very reliable in doing what it is capable of doing. It runs every time.

## Conclusion

A great temporary solution that provides a moderate amount of testing for my code. I'm still looking for an easy, reliable, and robust automated browser testing solution. Buster.js + BrowserStack or Testswarm look promising but they are still in early development. I'll be on the lookout though.