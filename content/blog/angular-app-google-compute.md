+++
categories = ["devops", "angularjs", "google compute engine"]
draft = false
showcomments = true
showpagemeta = true
date = "2014-08-16T10:26:47+08:00"
title = "Hosting an AngularJS app on an Google Compute Engine VM instance"
tags = []
slug = ""
comments = true

+++

Recently I was tasked with deploying an Angular app on Google Compute Engine(GCE). I spent a lot of time trawling the web for tutorials but they are scattered around as usual. With help from my co-worker, I managed to get it done. To prevent others from floundering about in this niche area, here’s my walkthrough.

First you have to create a project [here](https://console.developers.google.com/project). After that you’d have to create a VM instance. This step requires billing information as Google will charge you from the get-go.

Having created the VM instance, you will need to connect to it using `gcutil` tool from the terminal.

Follow the steps here to install `gcutil` and then to authenticate to GCE.

Since my project name is `foo-boulevard-629` and my instance name is `beechfork-2`, this would be my command to connect to the VM instance:

`gcutil --service_version="v1" --project="foo-boulevard-629" ssh --zone="asia-east1-a" "beechfork-2"`

After you are connected to the VM instance, you’ll need to install node.js to run the Angular app server. Since my VM instance is a Debian 7.0 (Wheezy), I followed this [Gist](https://gist.github.com/x-Code-x/2562576) to install node.

Be sure to check whether your installed `node` is working by running `node -v` before we proceed to the next step.

Next, `cd` into your preferred directory and clone your git repository containing your Angular app.

`git clone https://favmed.unfuddle.com/git/favmed_marathon/`

Install your Angular app dependencies using `npm install` and `bower install`.

At this time, we can start the server using Grunt by running `grunt serve`, however it is recommmend to use a daemon, e.g., `forever`, to make sure that the server runs continuously.

Let’s install `forever` first: `sudo npm install forever -g`

To use `forever`, we need to create a script that configures the port number and the folder to be served.

Create a file named `web.js` as shown below in your root directory of your Angular app:

```javascript
var gzippo = require('gzippo');
var express = require("express");
var logfmt = require("logfmt");
var app = express();

app.use(logfmt.requestLogger());
app.use(gzippo.staticGzip("" + __dirname + "/app"));

var port = Number(process.env.PORT || 5000);
app.listen(port, function() {
  console.log("Listening on " + port);
});
```

Before running the script, you need to install the required modules: `npm install gzippo express logfmt --save`

Run `forever web.js` to serve your Angular app.

In your VM instance page, you will see a external IP address. You can now access your Angular app at this link with the appropriate port number. Alternatively, you can configure your VM instance’s port number to be the same as the one that you set for your Angular app server, so that you can access your app without the port number.

Have fun.

