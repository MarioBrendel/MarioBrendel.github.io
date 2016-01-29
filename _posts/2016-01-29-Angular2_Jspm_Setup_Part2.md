---
layout: post
title: Angular 2 Starter Setup with JSPM, SystemJS and Typescript in atom (Part 2)
date: 2016-01-29
author: Mario Brendel
comments: true
categories: [Angular2-Setup]
tags: [Angular2-Setup-JSPM]
---
In this part we will talk more about the SystemJS-Hot-Reloader and how you can implement this awesome plugin.
Whenever you have problems with this tutorial, you can look up the code at [Github](https://github.com/MarioBrendel/Angular2-Jspm-Typescript-Atom-Seed) or contact me :).
<h2>Implementing SystemJS-Hot-Reloader</h2>
You may ask yourself what SystemJS actually is. Well you've already used it in your index.html file together with the config.js:

{% highlight html %}
[index.html]
...
<script>
  System.import('app').then(null, console.error.bind(console));
</script>
...
{% endhighlight %}

{% highlight javascript %}
[config.js]
...
  "app": { // all files within the app folder
  "main": "bootstrap", // main file of the package (will be important later)
  "format": "system", // module format
...
{% endhighlight %}

This nifty tool allows us to load any module format. Currently we are using ES6 module format as you can see in the line "format": "system". <br/><br/>
Now lets extend our [package.json](https://github.com/MarioBrendel/Angular2-Jspm-Typescript-Atom-Seed/blob/master/package.json) with the following lines:

{% highlight javascript %}
"devDependencies": {
    "chokidar-socket-emitter": "^0.4.2", // event source for systemjs-hot-reloader
    "http-server": "^0.8.5",             // basic http-server
    "open": "0.0.5"                      // will automaticly open our page on the given port
},
{% endhighlight %}

And run <pre><code>npm install --dev</pre></code>
After you have executed these commands run the following line
<pre><code>jspm install systemjs-hot-reloader</pre></code>
This is the one magic package that makes developing of Angular 2 applications such a breeze. It only reloads the components that got changed instead of
the entire app. For more information see [SystemJS-Hot-Reloader](https://github.com/capaj/systemjs-hot-reloader). <br/> <br/>

Now lets create a server.js file within our root folder that will combine our http-server with the chokidar-socket-emitter:
{% highlight javascript %}
'use strict'
const httpServer = require('http-server')
const open = require('open')

let cache = 3600
if (process.env.NODE_ENV === 'production') {
  console.log('running in production mode(with caching)-make sure you have "Disable cache (while DevTools is open)" checked in the browser to see the changes while developing')
} else {
  cache = -1
}
const server = httpServer.createServer({
  root: '.',
  cache: cache,
  robots: true,
  headers: {
    'Access-Control-Allow-Origin': '*',
    'Access-Control-Allow-Credentials': 'true'
  }
})

require('chokidar-socket-emitter')({app: server.server})

server.listen(9089)
open('http://localhost:9089')
{% endhighlight %}
To actually use the server.js file we'll implement the following lines in our package.json:

{% highlight javascript %}
"scripts": {
    "start": "node server"
  },
{% endhighlight %}

Last but not least we will update our index.html:

{% highlight html %}
<!doctype html>
<html>
<head>
  <title>My First Angular2 App</title>
  <script src="node_modules/angular2/bundles/angular2-polyfills.min.js"></script>
  <script src="jspm_packages/system.js"></script>
  <script src="config.js"></script>
</head>
<body>
  <app></app>
  <script>
    var readyForMainLoad
    if (location.origin.match(/localhost/)) {
      System.trace = true
      readyForMainLoad = System.import('systemjs-hot-reloader').then(function(HotReloader){
        hr = new HotReloader.default('http://localhost:9089');
      });
    }
    Promise.resolve(readyForMainLoad).then(function() {
      System.import("app").then(function(){ console.log("running") });
    });
  </script>
</body>
</html>
{% endhighlight %}

As soon as you run your app on localhost, the SystemJS-Hot-Reloader will be used. If you are running
the app in production the reloader won't be used.<br/>
<h2>Running The Application</h2>
Finally we will run our app with
<pre><code>npm start</pre></code>
And if everything went right, you should be able to hot reload your application.
<img src="../../../../../public/gifs/hot-reloading.gif">
<b>Note </b>that you might have cached the old version of the site. You may want to disable your cache and hit f5 to see the results.
<h2>Bundling Our Application</h2>
This was always the most annoying part for me. Bundling with gulp/webpack/... feels tiresome. Lets see how much scripts we have to write to bundle
with jspm.
 <pre><code>jspm bundle-sfx app build/app.js</pre></code>
... that's it. Seriously you only have to run this command and you'll get a script file which is completely independent of jspm/systemjs.
To test this we will change or index.html to:

{% highlight html %}
<!doctype html>
<html>
<head>
  <title>My First Angular2 App</title>
  <script src="node_modules/angular2/bundles/angular2-polyfills.min.js"></script>
  <script src="jspm_packages/system.js"></script>
  <script src="config.js"></script>
</head>
<body>
  <app></app>
  <script src="build/app.js"></script>
</body>
</html>
{% endhighlight %}

And you are good to go. If you want to now more about bundling, check out the [production workflow repo of jspm](https://github.com/jspm/jspm-cli/blob/master/docs/production-workflows.md).
<br/><br/>To get more informations about the package that we used, please go to: [Jspm](http://jspm.io/), [SystemJS](https://github.com/systemjs/systemjs) or [SystemJS-Hot-Reloader](https://github.com/capaj/systemjs-hot-reloader).
