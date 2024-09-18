+++
title = 'Express and Webpack together'
date = 2017-05-24T10:49:50+02:00
draft = false
+++
TL;DR Sample code on [Github](https://github.com/hfogelberg/pm2-demo).

One stumbling block when I first started using Vue was figuring out how to run a Node server in the same project as an application written in Vue JS. I searched all over the net but couldn’t really find anything and ended up with the code in two different projects. That’s fine of course, but I just felt it was a bit unaesthetic and messy. So the search continued!

[PM2](https://pm2.keymetrics.io/) to the rescue!
PM2 is a process management tool to start, stop and monitor Node JS applications. You need to install it globally running `npm install pm2 -g`. Once installed just type `pm2 start myApp.js` to kick off a Node app. You should check the docs for a complete list of how to use pm2, but the most important commands are:

```bash
pm2 start
pm2 list
pm2 stop
pm2 restart
```

But how does this help me?
A small example will make it a lot clearer than a long and winding explanation, so let’s pull up the command line and get going. Begin by installing pm2 globally if you haven’t done so already and then:

```bash
vue init webpack-simple pm2-demo
cd pm2-demo
npm install
npm install — save express
```

Open the project in your text editor and create a new file called index.js under the root and code up a minimal Hello World-ish api:

```
var express = require(‘express’);
var app = express();
app.get(‘/api’, (req, res) => {
    res.json({message: ‘Welcome to the Server’});
});
app.listen(8081, ()=>{
console.log(‘API listening on port 8081’);
});
```

We’ll just give it a quick check to see that there’s no typo or anything. Why not use pm2 to launch the application? At the command line just type in $pm2 start index.js. If everything is OK you should see a table like this:
{{< figure src="/images/pm2-console.png" title="" >}}

Just give the API a quick check: `curl -i -X GET http://localhost:8081/api`. And the stop the server `pm2 stop index`.

So how does this solve the problem?
All we have to do is edit the startup scripts in package.json. First of all create a new script to start the server:
```
"server": "pm2 start index.js --watch --name server",
```

The watch flag will watch your code for any changes and restart the server whenever you save your code — pretty much the same as nodemon. Try it out by typing $npm run server. Play around with calling the API with curl or Postman and change the message returned by the API.

The name flag allows you to set the name that is displayed in the list of running processes. The default is otherwise the filename. This is quite useful since it could be a bit confusing if you have 12 processes called index!

Second, add a script to start the webpack dev server using pm2:
```
"vue": "cross-env NODE_ENV=development pm2 start webpack-dev-server --name vue -- --open --hot",
```

Again, give it a try in the terminal by typing$npm run vue. You should see the good old Vue logo in your browser after a few seconds. And finally, to tie it all together, you just have to modify the dev script to start both the api and the Vue:
```
application: "dev": "npm run server | npm run vue",.
```

The final script will look like this:

```
“scripts”: {
“dev”: “npm run server | npm run vue”,
“vue”: “cross-env NODE_ENV=development pm2 start webpack-dev-server — name vue — — open — hot”,
“server”: “pm2 start index.js — watch — name server”,
“build”: “cross-env NODE_ENV=production webpack — progress — hide-modules”
},
```

If you have any processes running stop the using `pm2 stop all` and run the dev script `npm run dev`.

Enjoy!


The sample code is available on [https://github.com/hfogelberg/pm2-demo](Github).

