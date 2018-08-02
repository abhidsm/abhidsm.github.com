---
layout: post
title: Apache External Authentication
categories: Apache External Authentication
description: Apache external authentication for proxy requests. 
---

I have created a web application which is a middleware situated between the mobile app and the webservice provider. Some requests should be handled by the middleware and some other requests should be forwarded to the webservice provider.

We can add apache proxy to forward the requests to the webservice provider. But before forwarding the requests it should be validated. If we are validating from the application then we can’t use the apache proxy, so the validation has to be implemented within apache.

There is a custom module for apache to do this kind of external authentication called [mod-auth-external](https://code.google.com/p/mod-auth-external/). Where you can mention the path of an external authentication file. Whenever apache gets a request it will execute the validation file and based on the response (true/false) from validation file apache either continue with the request or return unauthorized.

<script src="https://gist.github.com/abhidsm/8fdb346fdf7bd1d4e3c2c47ca631aba3.js"></script>

