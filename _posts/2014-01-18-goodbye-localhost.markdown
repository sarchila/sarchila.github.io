---
layout: post
title:  "Goodbye localhost - Leveraging Node.js's process.env object for seamless deployment"
date:   2014-01-18 14:18:20
description: "When deploying a new project, one of the first potential triggers of head-scratching is if you have inadvertently used a hard-coded reference to your localhost (or 127.0.0.1) and/or port number in your server-side code."
---

When deploying a new project, one of the first potential triggers of head-scratching is if you have inadvertently used a hard-coded reference to your localhost (or 127.0.0.1) and/or port number in your server-side code. For instance, this can come about if you had written the full absolute path of your url to connect to your database.  When you deploy your project to a remote server somewhere (e.g. using [Heroku](https://www.heroku.com/) or [AWS](http://aws.amazon.com/)), these hard-coded references to localhost can cause many a gray hair.

## Web or API Server

The best strategy, in general, is to not specify the host for your web server or RESTful API, using only relative paths for any interaction you write on the server side.  One way to ensure this is you can literally search your entire server-side app folder for any instance of "localhost" or "127.0.0.1" and remove the host leaving only the relative reference (e.g. "http://localhost:3000/users" --> "/users").

## Port number and database server

For your port number or your database server host address, it helps to only specify them in only one place in your back-end app (e.g. in a config file) and, using Node's 'require' functionality, require that config file whenever you definitely need the absolute path.  One example where you need to use the absolute path is to connect to your database, as I'll demonstrate with a code snippet from my last project (where I used an [Express](http://expressjs.com) server built on [Node](http://nodejs.org/), using [Mongoose](http://mongoosejs.com/) as an ODM for [MongoDB](http://www.mongodb.org/)).  Instead of writing:
<br/>
<br/>
{% highlight javascript %}
//  file: ./server.js
  mongoose.connect('mongodb://localhost/app');
{% endhighlight %}
<br/>
 to connect to the 'app' database on your local MongoDB server, create a file in a config directory (maybe called dbconfig.js) where you export the url.
<br/>
<br/>
{% highlight javascript %}
//  file: ./config/dbconfig.js
  module.exports = {
    url: 'mongodb://localhost/app'
  };
{% endhighlight %}
<br/>
From then on, you can use that reference by importing that file as a module. So, instead of connecting to our database using a hardcoded reference to our localhost as we had done before, we would write:
<br/>
<br/>
{% highlight javascript %}
//  file: ./server.js
  var mongoConfig = require('./config/dbconfig');
  mongoose.connect(mongoConfig.url);
{% endhighlight %}
<br/>
This allows for flexibilty of usage for when your host changes (such as when you deploy).  The same strategy can be used for your port number.  You can create a file in your config folder called serverConfig.js which could include:
<br/>
<br/>
{% highlight javascript %}
//  file: ./config/serverConfig.js
  module.exports = {
    port: 3000
  };
{% endhighlight %}
<br/>
{% highlight javascript %}
//  file: ./server.js
  var express = require('express');
  var app = express();
  var port = require('./config/serverConfig').port;
  app.listen(port);
{% endhighlight %}
<br/>
But, as you might have noticed, this doesn't actually resolve the problem at hand; we still have a hard-coded reference to our local host and port number. So far, we have just removed many hard-coded references and replaced them with a single one in a config file, which we can import when needed.  This is where [Node's process object](http://nodejs.org/api/process.html#process_process) comes in.

## process.env

The <code>process</code> object is a global object that is available whenever you have a Node instance running.  It contains a <code>env</code> property that is an object populated with properties of the user environment.  You can easily get a sense of the information contained by running Node in your terminal and then inspecting the object:
<br/>
<pre><code>
//  run node in terminal
$ node
\> process.env
</pre></code>
<br/>
When deploying (to Heroku, for instance), you can leverage this object to assign and retrieve environment-specific values.  This can include the port you wish to listen on as well as your database url.  So now, you would only have to change your config files to say: 
<br/>
<br/>
{% highlight javascript %}
//  file: ./config/serverConfig.js
  module.exports = {
    port: process.env.PORT || 3000 
  };
{% endhighlight %}
<br/>
{% highlight javascript %}
//  file: ./config/dbconfig.js
  module.exports = {
    'url': process.env.MONGOHQ_URL || 'mongodb://localhost/app'
  };
{% endhighlight %}
<br/>
This makes the specified port and database url the default or fallback values. If I set the PORT or the MONGOHQ\_URL properties on the process.env object of my deployment environment, those values will be used instead. Of course, the MONGOHQ\_URL property is specfic to the fact that I used the [MongoHQ add-on for Heroku](https://addons.heroku.com/mongohq) in my app.  Yours may be different if you decide to use a different database, cloud host, and/or add-on module.
<br/>
<br/>
By following these guidelines while building your app, you should save yourself many a headache when it comes time to deploy.  Or at the very least, the source of your headaches will be somewhere else! :)
