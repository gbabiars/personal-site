---
title: "Using Grunt As Your Front End Dev Server"
description: "Need for Front End Seperation For many large development projects, there is a need to seperate the front and back end development into two different projects. The benefit here…"
pubDate: "2013-08-04T00:00:00"
---

## Need for Front End Seperation

For many large development projects, there is a need to seperate the front and back end development into two different projects. The benefit here for the front end team is that they can work in html, javascript and css without having to worry about compiling new builds. One issue that must be faced when doing this is making sure the local development enviroment can both server local static files and also connect to services that ajax calls will be connecting to. For us, this meant serving up our backbone app while connecting to an api on one of several VMs. Originially, we used XAMPP on each devs machine to serve up the files. The downsides of this are that it requires each machine to be manually configured and the config is not stored in the projects version control repositiory.

## Using Grunt Instead

Grunt's wide range of makes it easy to create a dev server to both serve up our static files as well as proxy to our services. Using [grunt-contrib-connect](https://github.com/gruntjs/grunt-contrib-connect), we created a simple task to serve our project's static files.

    connect: {
      'static': {
        options: {
          hostname: 'localhost',
          port: 8001
        }
      }
    }

This task just simply serves up static files to localhost:8001 using the folder our Gruntfile.js is in as the root directory. While this was easy enough, we still need to get it to connect to our api. In order to do this, we need a proxy server to proxy our local service calls to the correct remote server. Luckily there is a great project called [grunt-connect-proxy](https://github.com/drewzboto/grunt-connect-proxy) that allows us to modify our connect configuration to specify which service calls we want to proxy and to where. Here is the resulting modifications we've made.

    connect: {
      'static': {
        options: {
      	  hostname: 'localhost',
      	  port: 8001
        }
      },
      server: {
        options: {
          hostname: 'localhost',
          port: 8000,
          middleware: function(connect) {
            return [proxySnippet];
          }
        },
      	proxies: [
          {
          	context: '/svc',
          	host: 'myserviceshost',
          	port: 8080
          },
          {
          	context: '/api',
          	host: 'myapihost',
          	port: 9080
          },
          {
            context: '/',
            host: 'localhost',
            port: 8001
          }
    	]
      }
    }

At the top of our Gruntfile, we declare our proxySnippit:

{% codeblock lang:js %}
var proxySnippet = require('grunt-connect-proxy/lib/utils').proxyRequest;
{% endcodeblock %}

There are a few things to notice in our config. First, we've added a second connect task called server. This will run at localhost:8000 and will encompass both our static files and our service urls. We add a middleware function to make sure we run the proxySnippet provided by grunt-connect-proxy. We've added a proxies array which defines what urls are proxied where. We've defined three proxies. Calls to http://localhost:8000/svc* will be proxied to http://myserviceshost:8080/svc. Calls to http://localhost:8000/api* will be proxied to http://localhost:9080/api. Finally, calls to http://localhost:8000/* will be proxied to http://localhost:8001. This last proxy makes sure that our static files are served from our "static" connect task. Notice that the context property in each proxy tells us which urls to look for. I have noticed that order is important here, going from highest priority to lowest. We define '/' last so that the other service calls will take priority.

Now that we have configured our connect task, we need to register a new task that will put this all together. Here is what it will look like:
{% codeblock lang:js %}
grunt.registerTask('server', ['connect:static', 'configureProxies:server', 'connect:server', 'watch']);
{% endcodeblock %}
Running `grunt server` will start our static server at localhost:8001, configure our proxies defined in our proxies options array, then start the server at localhost:8000. Lastly we add a watch task using [grunt-contrib-watch](https://github.com/gruntjs/grunt-contrib-watch) to keep our server running. Now if we go to http://localhost:8000, we will have our development server with our local html/javascript/css files while still using the correct remote service calls! And we can commit this to source control so no manual configuration is needed.

## Taking it Further

While connecting to remote services was our main goal, we can also create new configurations to cover other use cases. For example, during new development our remote server may not have services that we need. Rather than wait for the backend to implement these to start our development, we instead point to fake servers running locally (using node). Using the same configuration logic as we had for our server task, we can create a fakeServer task that proxies to our fake local services.

    // in the connect task...
    fakeServer: {
    	options: {
    		hostname: 'localhost',
    		port: 8000,
    		middleware: function(connect) {
    			return [proxySnippet];
    		}
    	},
    	proxies: [
    		{
    			context: '/svc',
    			host: 'localhost',
    			port: 8080
    		},
    		{
    			context: '/api',
    			host: 'localhost',
    			port: 9080
    		},
    		{
    			context: '/',
    			host: 'localhost',
    			port: 8001
    		}
    	]
    }

    // we register the fakeServer task
    grunt.registerTask('fakeServer', ['connect:static', 'configureProxies:fakeServer', 'connect:fakeServer', 'watch']);

Now if we run `grunt fakeServer` we can use the fake services to develop locally, then when the real backend services are ready, we can test against them. The added benefit here is that we may want to test many different use cases such as an endpoint returning no data or error handling. This is really easy to do on our fake server without affecting production code.

## Putting It All Together

To see the full example, check out this [gist](https://gist.github.com/gbabiars/6149943).
