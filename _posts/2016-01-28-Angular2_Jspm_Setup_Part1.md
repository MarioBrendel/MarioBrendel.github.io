---
layout: post
title: Angular 2 Starter Setup with JSPM, SystemJS and Typescript in atom (Part 1)
date: 2016-01-28
author: Mario Brendel
comments: true
categories: [Setup]
tags: [Angular2-Setup-JSPM]
---
This blog post will be about the setup of Angular 2 with jspm and systemjs. In this post we will use the angular2 version 2.0.0-beta.1. This version might not work correctly for the IE. If you want to develop for the internet explorer you may want to use beta.0.

<h2>Prerequisites</h2>
<ul>
<li><a href="https://www.npmjs.com/package/npm">NPM version >= 2</a></li>
<li><a href="https://nodejs.org/en/download/">NodeJS version >= 4</a></li>
</ul>
<h2>Setting Up Atom</h2>
If you haven't already downloaded atom, please navigate to: [atom.io](https://atom.io).
Next you have to install some packages within your editor. Hit 'ctrl + ,' to manover to your settings. Then select install and download: [Atom-Typescript](https://atom.io/packages/atom-typescript). As you can see there are a lot features that comes with the package. My personal favorite is the autocompletion and live error analysis.

<h2>Setting up the Project</h2>
At first we create a folder called Angular2-JSPM-Setup and navigate our terminal to the folder. Within our terminal we run the follow commands:

{% highlight bash %}
npm install jspm -g
jspm init -y
{% endhighlight %}

The npm install is used to get jspm which will install our frontend package manager. For more information see: [JSPM](http://jspm.io/)
<br />JSPM init will initialize our project. The -y flag is used to answer all the init questions with yes. If you want you can call jspm init without the -y flag and discover, what jspm lets you configure.</pre><br /><br />
After you have executed these commands, create an app folder within you root folder (Angular2-JSPM-Setup). Your project should now look like this: ![folder structure]({{ site.url }}/public/images/2016-01-28-Setup/Setup_after_JSPMInit.PNG)
<br/>
Now we will change the config.js a little bit to make it compatible with our following typescript files. To do that copy the following code and paste it  in the System.config:

{% highlight javascript %}
typescriptOptions: {
    "tsconfig": true // indicates that a tsconfig exists that should be used
  },
  packages: {
    "app": { // all files within the app folder
      "main": "bootstrap", // main file of the package (will be important later)
      "format": "system", // module format
      "defaultExtension": "ts", // default extension of all files
      "meta": {
        "*.ts": { // all ts files will be loaded with the ts loader
          "loader": "ts"
        },
      }
    }
  },
{% endhighlight %}

File: [config.js](https://github.com/MarioBrendel/Angular2-Jspm-Typescript-Atom-Seed/blob/master/config.js)<br/><br/>
And finally we will create a [tsconfig.json](https://github.com/Microsoft/TypeScript/wiki/tsconfig.json) file within our root folder. This file is used to give the typescript compiler some informations about our environment.

{% highlight javascript %}
{
 "compilerOptions": {
    "target": "es5",                /* target of the compilation (es5) */
    "module": "system",             /* System.register([dependencies], function) (in JS)*/
    "moduleResolution": "node",     /* how module gets resolved (needed for Angular 2)*/
    "emitDecoratorMetadata": true,  /* needed for decorators */
    "experimentalDecorators": true, /* needed for decorators (@Injectable) */
    "noImplicitAny": false          /* any has to be written explicity*/
  },
  "exclude": [   /* since compiling these packages could take ages, we want to ignore them*/
    "jspm_packages",
    "node_modules"
  ],
  "compileOnSave": false        /* on default the compiler will create js files */
}
{% endhighlight %}

File: [tsconfig.json](https://github.com/MarioBrendel/Angular2-Jspm-Typescript-Atom-Seed/blob/master/tsconfig.json)<br/><br/>
<b>Note: </b>The comments in the tsconfig file might give you an error, so you better delete them.
<br/><br/>To get all the packages we need for development, we will use jspm install.
To find out what packages are currently available with the jspm installer
you can use the [jspm-registry](http://kasperlewau.github.io/registry/#/). So now lets run some more code in our terminal:
<pre><code>jspm install ts <br/>jspm install angular2</pre></code>
The first command will install the typescript plugin which is used for our ts loader, that you can see above.
<br/><br/>
Normally this is everything you need for your project setup, but there is currently an error with the [module resolution](https://github.com/Microsoft/TypeScript/issues/6012).
So we need to add the following lines to our package.json file:

{% highlight javascript %}
"dependencies": {
    "angular2": "2.0.0-beta.1",
    "es6-promise": "^3.0.2",
    "es6-shim": "^0.33.3",
    "reflect-metadata": "0.1.2",
    "rxjs": "5.0.0-beta.0",
    "zone.js": "0.5.10"
}
{% endhighlight %}

File: [package.json](https://github.com/MarioBrendel/Angular2-Jspm-Typescript-Atom-Seed/blob/master/package.json)<br/><br/>
Now call npm install within your terminal and all dependencies will be downloaded for your project. Sadly we have duplicated all our angular2 dependencies, but this is necessary since the live error analysis would throw errors as soon as we import a module from the angular2 package.
<br/><b>Note: </b>Our code would also run without the angular2 files from the npm install, <b>BUT</b> the live error analysis will show an error for every angular2 module we want to use. This could be really confusing so I would recommend to install angular2 via npm too. I hope that this bug will be solved in the future because jspm is probably the best frontend package manager. If you are from the java world it mostly feels like maven.
<h2>Writing Some Code</h2>
Now that we've got everything wired up we will write some code. At first we'll create a index.html file at our root folder with the following code:

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
  <!-- this will be our entry component for the application -->
  <app></app>
  <script>
    System.import('app').then(null, console.error.bind(console));
  </script>
</body>
</html>
{% endhighlight %}

Next we will create the bootstrap.ts file in the app directory:

{% highlight javascript %}
import { bootstrap } from 'angular2/platform/browser';
import {AppComponent} from "./app.component";

bootstrap(AppComponent);
{% endhighlight %}

and the corresponding app.component.ts file

{% highlight bash %}
import {Component} from 'angular2/core';
@Component({
  selector: 'app',
  template: `
  <p>Hello World</p>
  `
})
export class AppComponent {
  constructor() { }
}
{% endhighlight %}

Finally our folder structure should look something like this:
![final setup]({{ site.url }}/public/images/2016-01-28-Setup/Setup_final(part 1).PNG)
<h2>Running The Application</h2>
To run our Application we execute the following commands:
<pre><code>npm install live-server -g <br/>live-server</pre></code>
This will start our application and print hello world on your browser. If you had any issues with this tutorial feel free to contact me.
<br/></br>In the next part of this tutorial we will use SystemJS Hot reloader, so that we
don't have to wait 5 seconds until every change to our app is published.
Furthermore we will use the bundling of jspm to generate an independent js file of our application.
