---
layout: post
title: Front End App CORS issue in development environment 
categories: Front End App CORS issue in development environment 
description: When we develop a Front End App which need access to the web services will face the Cross Origin Resource Sharing issue. This post is about how to solve this issue.
---

When you are developing a Front End Application or Single page
application using Backbone.js, Angular.js or any other MV* frameworks,
you might have faced the issue with CORS. You will face this issue if
you are accessing the hosted web services (API) from your development
environment because the domains are different.

We can solve the CORS issue in different ways. First method is
[JSONP](http://en.wikipedia.org/wiki/JSONP). To implement JSONP you
need the access to modify the web services. Because to implement the
JSONP the web service has to return the json data wrapped in a
function. You can do this only if you have the rights to change the
web service.

If you don't have the right to change the web services, the next
method to solve CORS issue is to set the Header [Access Control Allow
Origin](http://enable-cors.org/server_apache.html) in the server. You
can set the Origin to multiple domains to support access from multiple
domains. For this you need access to server to modify the apache/nginx
virtual host configuration. But what if you don't have access to the server at all?

The next and the best solution is to implement [reverse
proxy](http://www.apachetutor.org/admin/reverseproxies) in the
development environment. Here is a sample virtual host file which has
reverse proxy implemented: 

    <VirtualHost *:80> 
        ServerName front_end_app.abhilash-desktop.com 
        DocumentRoot /home/abhilash/Projects/front_end_app/public 
        <Directory /home/abhilash/Projects/front_end_app/public> 
           AllowOverride all    
           Options -MultiViews  
        </Directory> 
           
        LoadModule proxy_module modules/mod_proxy.so 
        ProxyRequests Off 
      
        <Proxy *> 
          Order deny,allow 
          Allow from all 
        </Proxy> 
      
        ProxyPass /api http://abhidsm.com/api 
        ProxyPassReverse /api http://abhidsm.com/api 
                 
    ErrorLog /var/log/apache2/error.log 
                   
    # Possible values include: debug, info, notice, warn, error, crit, 
    # alert, emerg. 
    LogLevel warn 
         
    CustomLog /var/log/apache2/access.log combined 
    </VirtualHost>

From your app if you start accessing the /api URL it will be accessed
from the proxy URL. I think this is the easiest and best method to
solve the CORS issue in development environment.
