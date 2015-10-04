---
layout: post
title: Apache External Authentication
categories: Apache External Authentication
description: Apache external authentication for proxy requests. 
---

I have created a web application which is a middleware situated between the mobile app and the webservice provider. Some requests should be handled by the middleware and some other requests should be forwarded to the webservice provider.

We can add apache proxy to forward the requests to the webservice provider. But before forwarding the requests it should be validated. If we are validating from the application then we can’t use the apache proxy, so the validation has to be implemented within apache.

There is a custom module for apache to do this kind of external authentication called [mod-auth-external](https://code.google.com/p/mod-auth-external/). Where you can mention the path of an external authentication file. Whenever apache gets a request it will execute the validation file and based on the response (true/false) from validation file apache either continue with the request or return unauthorized.

Sample virtual_host.conf file:

    LoadModule authnz_external_module modules/mod_authnz_external.so
    <VirtualHost *:80>
    DocumentRoot "/var/www/application_name"
    ServerName application_name.com
    AddExternalAuth auth /var/www/application_name/validate.php
    SetExternalAuthMethod auth environment
    <Directory "/var/www/application_name">
       Options Indexes +FollowSymLinks MultiViews +ExecCGI
       AllowOverride None
       Order allow,deny
       Allow from all
    </Directory>
    <Location ~ "^/api/(.*)$">
       AuthName "AppName"
       AuthType Basic
       AuthBasicProvider external
       AuthExternal auth
       require valid-user
    </Location>
    ProxyRequests Off
    ProxyPreserveHost On
    ProxyPass /api/ https://192.168.1.200/Webserver/
    ProxyPassReverse /api/ https://192.168.1.200/Webserver/
    <Proxy *>
       Order deny,allow
       Allow from all
    </Proxy>
    </VirtualHost>

Sample external authentication file (validate.php):

    #!/usr/bin/php
    <?php
    $host = "127.0.0.1";
    $user = "user";
    $password = "password";
    $database = "database";
    $link = mysql_connect($host, $user, $password);
    $selected = mysql_select_db($database, $link);
    $username = getenv('USER');
    $password = getenv("PASS");
    $sql = "SELECT * FROM table_name WHERE username= '$username' AND password = '$password' AND status = 1";
    $result = mysql_query($sql, $link);
    $count = mysql_num_rows($result);
    if ($count > 0) {
      exit(0);
    } else {
      exit(1);
    }  
    ?> 
