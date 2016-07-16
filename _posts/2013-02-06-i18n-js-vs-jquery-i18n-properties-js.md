---
layout: post
title: "i18n.js vs jquery.i18n.properties.js"
tags: require.js, javascript, i18n.js, jquery
---
Internationalization is a big deal these days, but implementing it is still a
pain sometimes. With the rise in popularity of JavaScript-heavy applications
using tools like [Require.js](http://requirejs.org/ "Require.js"), [Backbone.js](http://backbonejs.org/ "Backbone.js"),
[Handlebars.js](http://handlebarsjs.com/ "Handlebars.js"), etc., one naturally
seeks a solution to the internationalization problem that fits into the
paradigms of these tools. I kept a couple of points in mind while searching for
a solution:

- <span style="line-height: 13px;">The tool introduces no new dependencies into the build process (e.g. I don't want to have to install Ruby)</span>
- The tool allows storage of i18n text in simple, easy-to-use, and preferably super common formats (e.g. JSON, or Java's .properties)
- The tool has a small footprint on projects
- The tool lets me to easily access the i18n text inside my templates
I found a number of tools out there, but two jumped out at me: i18n.js (for Require.js) and jquery.i18n.properties.js

### i18n.js

A plugin for Require.js, i18n.js loads i18n text files as dependencies. i18n
files are .js files that use the "define" wrapper function. The default i18n
file (with no locale specified) could look like this:

path/to/i18n/files/i18n_module_A.js
```js
define({
  "root": {
    "someText": "A short sentence",
    "otherText": "pink pants"
  },
  "ru-RU":true,
  "fr-fr":false
});
```
The "root" is the default text. "ru-RU":true tells i18n.js that the application
supports Russian with region code RU, therefor i18n.js should expect to be able
to find files in an ru-RU folder that provide the Russian text. Therefore I
should also have the file:

path/to/i18n/files/ru-RU/i18n_module_A.js
```js
define({
  "someText": "короткое предложение",
  "otherText": "розовые штаны"
});
```
"fr-fr":false tells i18n.js that "fr-fr" is not supported, so even if the
browser's locale is fr-fr, i18n.js will use the default text in "root". If
module_A.js needs the text in i18n_module_A.js, then it loads the text like a
dependency:

module_A.js
```js
define([
  'someDep',
  'i18n!path/to/i18n_module_A'
], function (SomeDep, i18n_module_A) {

  // i18n_module_A is a JavaScript Object.

  // Prints "a short sentence" if the browser's locale is not supported by the application
  // Prints "короткое предложение" if the browser's locale matches "ru-RU"
  console.log(i18n_module_A.someText);

  // Prints "pink pants" if the browser's locale is not supported by the application
  // Prints "розовые штаны" if the browser's locale matches "ru-RU"
  console.log(i18n_module_A.otherText);
});
```
i18n.js also supports country codes only (e.g. "ru"):

path/to/i18n/files/ru/i18n_module_A.js
```js
define({
  "someText": "короткое предложение",
  "otherText": "розовые штаны"
});
```
Loading the i18n text as dependencies proves to be pretty nifty. During the r.js
optimization the default text files get in-lined into the build, but there will
always end up being HTTP requests for specific country/region files. In the
Require.js callback function i18n text is loaded into a JavaScript Object
automatically, which can simple be passed to a templating engine's render
function. The template can then pick text off the object. It made for a very
small amount of code to actually implement internationalization via i18n.js. You
simply depend on an i18n file and pass the resulting JavaScript Object to your
template. Simple.

However, i18n.js requires the i18n files to be in some obscure format. If you
don't need any server-side code (that's not Node.js) to read these files then
it's not a big deal, but languages like Java already use the well-known
.properties format, with which translation companies are familiar and have tools
to parse. In my case all of my i18n files are in the .properties format, so I
had to write scripts to convert the .properties files to the .js files
consumable by i18n.js, which was a hassle and an unnecessary addition to the
build process.

Another thing against i18n.js is that it does not support parameterization of
strings. For example, if I need to be able to insert a user's name into a
sentence, then I have to break the sentence up into two pieces. Very messy.

**Final thoughts on i18n.js**: Trivial to setup with Require.js. Non-trivial to
make it work with other file format's and pre-existing standards. Lack of
parameterization is a problem.

### jquery.i18n.properties.js

Who wouldn't jump at the chance to add another jQuery plugin to their site?
Luckily it's only 4.3KB minified. The convenience of this plugin, at least for
me, is that it directly consumes .properties files. Now my Java services can
consume the same i18n files as the frontend.

Messages.properties
```
# This line is ignored by the plugin
msg_hello = Hello
msg_world = World
msg_complex = Good morning {0}!
```
Messages_ru_RU.properties
```
# This line is ignored by the plugin
msg_hello = привет
msg_world = мир
msg_complex = доврое утро {0}!
```
.properties files are loaded with an Ajax call:

Loading a .properties file
```js
// This will initialize the plugin
// and display "Good morning John!"
jQuery.i18n.properties({
  name:'Messages',
  path:'path/to/i18n/files/',
  mode:'map',
  language:'ru_RU',
  callback: function() {
    // I specified mode: 'map' because the 'vars' mode
    // will add a lot of global JS vars to the page.

    // Accessing a simple value through the map
    $.i18n.prop('msg_hello');

    // prints 'привет'
    alert($.i18n.prop('msg_hello'));
  }
});
```
Another great plus for this plugin is that is supports parameterization:

parameterization
```js
// prints доброе утро John!
alert($.i18n.prop('msg_complex', 'John'));
// does the same as above
alert($.i18n.map.msg_complex('John'));
```
Initially I was worried that using this plugin would result in a lot more lines
of code in the JavaScript, as I thought that I would need to manually extract
each line of text from the .properties files. This plugin does not put all of
the i18n text into a JavaScript object like the i18n.js plugin does. However, it
was rather simple to write a helper for Dust.js that allows me to access i18n
strings loaded by jQuery directly in my templates:

Dust.js helper
```js
dust.helpers.i18n = function (chunk, ctx, bodies, params) {
  var key = dust.helpers.tap(params.key, chunk, ctx);
  return chunk.write($.i18n.map[key]);
};
```
and then in the template:

Accessing i18n strings in the template
```html
<h1>
  {@i18n key="msg_hello" /}
</h1>
```
**Final thoughts on jQuery.i18n.properties.js**: I really appreciate this
plugin's support for parameterization of strings. More importantly, it consumes
.properties files which is excellent.

### Conclusion

With the help of the Dust.js helper function it ends of being much more
convenient to work with the jQuery.i18n.properties.js plugin, rather than the
i18n.js plugin for Require.js. I haven't looked deeply into other solutions for
internationalization, but the jQuery plugin meets my needs just fine.
