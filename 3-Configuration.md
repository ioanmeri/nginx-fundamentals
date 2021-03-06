# Configuration

## Understanding Configuration Terms

It's important to understand the terminology used in nginx configuration files.

The 2 main configuration terms is **context** and **directive**

**Directive**: is specific configuration options that get set in the configuration files and consist of a **name** and a **value**.

* e.g. 
```
server_name mydomain.com
``` 

**Context**: is sections within the configuration where directives can be set for that given context. Essentially is the same as scope. Contexts are also nested and inherit from their parents. The topmost context is the **configuration file** itself. It's called **Main Context**

* The Main context is where we configure global directives that apply to the master process.

* e.g. 
```
http {
  index index.html index.html index.php;

  include mine.types;

  server {
    listen 80;
    server_name mydomain.com
    access_log /var/log/mydomain.com.access.log main;
    root html;

    location /some_path {
      add_header header_name header_value;
    }
  }
}
```

Important contexts: HTTP (for anything http related), server (where we define a virtual host), location (for matching URI locations on incoming requests to the parent server context)


##  Creating a Virtual Host

We 'll finally get rid of the default nginx holding page by creating a **basic virtual host to serve static files** from a directory on our server.

1. create a directory /sites/demo

2. upload a very simple single page website via **FileZilla**
demo directory contains an .html, .css, and thumb.png
To connect:
Host: sftp://167.172.62.98, user: root, port: 22

3. edit configuration files: /etc/nginx/nginx.conf

Open with a text editor and delete anything inside http context, the top of the main context, and everything inside events context:

```
events {}

http {
  
}
```

event though we **won't be** adding anything to the **event context**, we do need to keep it for the configuration to be considered valid.

Define a **virtual host** with a server context.

* a virtual host being a server context or server block is essentially responsible for listening on a port, typically 80 for http or 443 for https.

```
http {

    server {

        listen 80;
    }
}
```
you could leave out this listen directive in which case nginx will assume port 80 by default, but it is considered good practice to include it anyway.

```
http {

    server {

        listen 80;
        server_name 167.172.62.98;
    }
}
```

server_name: domain, subdomain or IP address for which the context server exists.

```
http {

    server {

        listen 80;
        server_name 167.172.62.98;

        root /sites/demo;
    }
}
```
we are setting the root path to **/sites/demo** in order to serve static requests from that demo directory.

4. save that
and now for these changes to take effect will have to reload nginx.

We use the reload command over the restart command as this way we prevent any downtime.

```
systemctl reload nginx 
```

When nginx is not working as expected, first step:
```
nginx -t
```

> Caution! the sites directory should be created inside the top root directory! 


## Location Blocks

The most used context in any nginx configuration.
It's how will define and configure the behaviour of specific URI's or requests.

They intercept a request based on its value and then doing something other than just trying to serve a matching file relative to the root directory.


### Prefix match with /

much anything starting with /greet, like /greeting or /greeting/more

```
http {

    include mime.types;

    server {

        listen 80;
        server_name 167.172.62.98;

        root /sites/demo;

        location /greet {
            return 200 'Hello from NGINX "/greet" location';
        }
    }
}
```

### Exact Match with = /

```
location = /greet {
    return 200 'Hello from NGINX "/greet" location';
}
```

### REGEX match with ~ /
/greet4, greet0, greet9

Tilda modifier is case sensitive
```
location ~ /greet[0-9] {
    return 200 'Hello from NGINX "/greet" location';
}
```


### REGEX match case insensitive with ~* /
/greet4, greet0, greet9

~* modifier is case insensitive
```
location ~* /greet[0-9] {
    return 200 'Hello from NGINX "/greet" location MATCH CASE INSESITIVE';
}
```

REGEX >>> PREFIX MATCH


### Preferential Prefix match with ^~ /

Essentially the same as the basic prefix modifier only more important than a regular expression match.

```
location ^~ /Greet2 {
    return 200 'Hello from NGINX "/Greet2" location MATCH CASE INSESITIVE';
}
```

### The Order of priority
It's important to understand the order of priority in which nginx matches requests

1. Exact Match,               = uri
2. Preferential Prefix Match  ^~ uri
3. REGEX Match,               ~* uri
4. Prefix Match               uri


## Variables

The nginx configuration syntax closely resembles that of a programming language. Implementing in some ways the concept of scope, includes and variables and conditionals

### Forms of Variables

* Configuration Variables
  * set $var 'something';

* NGINX Module Variables
  * $http, $uri, $args
  * [Alphabetical index of variables](http://nginx.org/en/docs/varindex.html)


#### Nginx variables

```
http {

    include mime.types;

    server {

        listen 80;
        server_name 167.172.62.98;

        root /sites/demo;

        location /inspect {
            return 200 "$host\n$uri\n$args";
        }
    }
}
```

If i visit http://167.172.62.98/inspect?name=ray, the output is: 
167.172.62.98
/inspect
name=ray

To get a **specific arg**:
Based on the query string, nginx compiles a named variable for each parameter, prefixed with arg:

```
location /inspect {
    return 200 "Name: $arg_name";
}
```

For http://167.172.62.98/inspect?name=ray, we get:

**Name: ray**


### Basic Conditionals

Most commonly will be used in conjuction with these variables.

> Note that the use of nginx congitionals inside location contexts is highly discouraged, as this can lead to some very unexpected behaviour.


```
http {

    include mime.types;

    server {

        listen 80;
        server_name 167.172.62.98;

        root /sites/demo;

        # Check static API key
        if ( $arg_apikey != 1234 ){
            return 401 "Incorrect API Key";
        }

        location /inspect {
            return 200 "Name: $arg_name";
        }
    }
}
```

### Configuration variables

Our Own Variables:

```

server {

    listen 80;
    server_name 167.172.62.98;

    root /sites/demo;

    set $weekend 'No';

    # Check if weekend
    if ( $date_local ~ 'Saturday|Sunday' ){
        set $weekend 'Yes';
    }

    location /is_weekend {
        return 200 $weekend;
    }
}
```


## Rewrites

Rewrites or sipmly redirects in some cases.

Two **directives** for rewriting requests:

* rewrite
  * pattern URI
* return 
  * status URI

When we are using return directive and the response code is a 300 variant, which is for redirects,
e.g.
```
return 307 /some/path;
```
return directive changes in it then except a URI as the second parameter

Let's say we want to serve the image /thumb.png for /logo:

### 307 Temporary redirect with return
```
location /logo {
    return 307 /thumb.png;
}
```

But the url changed **from /logo to /thumb.png**, which is essentially **the main difference between rewrites and redirects**

A redirect simply tells the client performing the request, where to go instead.

A rewrite on the other hand, mutates the URI internally.


### Rewritting

When a URI is rewritten it also gets re-evalutated by nginx as a completely new request. Meaning:

```
server {

    listen 80;
    server_name 167.172.62.98;

    root /sites/demo;

    rewrite ^/user/\w+ /greet;

    location /greet {
        return 200 "Hello user";
    }
}
```
/user/john, at this point the rewritten greet URI starts from the top again and will get re-evaluated again. The re-evaluation makes rewrites very powerful but also requires more system resources than a return.

#### Capture Groups

Another important and powerful feature of rewrites is the **ability to capture certain parts of the original URI**, using standard **regular expression capture groups**. Simply wrap tha part in braces and can be accessed as $1. If we have a second capture group, we can access that value as $2.

```
server {

    listen 80;
    server_name 167.172.62.98;

    root /sites/demo;

    rewrite ^/user/(\w+) /greet/$1;

    location /greet {
        return 200 "Hello user";
    }

    location = /greet/john {
        return 200 "Hello John";
    }
}
```

#### Passing Optional Flags

**Last Flag**

```
server {

    listen 80;
    server_name 167.172.62.98;

    root /sites/demo;

    rewrite ^/user/(\w+) /greet/$1;
    rewrite ^/greet/john /thumb.png;

    location /greet {
        return 200 "Hello user";
    }

    location = /greet/john {
        return 200 "Hello John";
    }
}
```

What the last flag allows us to do is essentially tell nginx not to allow a URI to be rewritten anymore:

Make sure it is the last time it gets rewritten, meaning it should skip over the second rewrite.
Request the same URI with reload.
```
rewrite ^/user/(\w+) /greet/$1 last;
```

## Try Files & Named Locations

Try files can be used in the server context, so applying to all incoming requests, or inside a location context.

What try_files allows us to do is have nginx check for a resource to respond with in any number of locations, relative to the root directory with a final argument that results in a rewrite and re-evaluation.

```
try_files path1 path2 final;
```
What this directive is doing all the time being in the server context, is checking whether: 
**/sites/demo/thumb.png** exists, and if it does serve it.

If however this first argument doesn't exist, move on and try next one, and so on...

```
server {

    listen 80;
    server_name 167.172.62.98;

    root /sites/demo;

    try_files /thumb.png /greet;

    location /greet {
        return 200 "Hello user";
    }

}
```

In our case, thumb.png definitely exists relative to the root directory so this should be served for all requests. 

When try_files reaches its last argument, that argument is treated as an internal rewrite. Meaning, in this case the rewritten request will be also re-evaluated.


### try_files and variables

```
server {

    listen 80;
    server_name 167.172.62.98;

    root /sites/demo;

    try_files $uri /greet /friendly_404;

    location /friendly_404{
        return 404 "Sorry, that file could not be found";
    }

    location /greet {
        return 200 "Hello user";
    }

}
```

check the error code by:
```
curl -I 167.172.62.98/index3
```

### Named Locations

Simply means assigning a name to a location context. And using a directive such as try files, use that location by it's name, ensuring no re-evaluation has to happen on that final argument but instead just a definite call to the named location.


```
try_files $uri /greet @friendly_404;

location @friendly_404{
    return 404 "Sorry, that file could not be found";
}
```


## Logging

nginx provides us two log types: **error logs** and **access logs**.

Error logs for anything that failed or didn't happen as expected.

Access logs for all requests to the server.

Logs allows us to track down errors or even identify malicious users.

In our case we specified the path to the log files when we configured nginx.

### Check access logs

```
cd /var/log/nginx/
echo '' > access.log
echo '' > error.log
ls -l
```

visit /thumb.png

```
cat access.log
```

404 is a perfectly fine response and by no means an error. It is not logged in the error file!


### Custom Access log
```
location /secure {
  access_log /var/log/nginx/secure.access.log;
  return 200 "Welcome to secure area";
}
```

or log in both. 
Log directives are of the few directives that you can use multiple times like this.

```
location /secure {
  access_log /var/log/nginx/secure.access.log;
  access_log /var/log/nginx/access.log;
  return 200 "Welcome to secure area";
}
```

### Disable access log

Disable logging for sites receiving very high traffic. It reduces server load and keeps log files smaller.

```
location /secure {
  access_log off;
  return 200 "Welcome to secure area"
}
```

Disable the majority of access logs, and leave the error logs in place.


## Inheritance & Directive types

As in programming languages, an nginx context inherits configurations from its parent contexts.
Top to bottom.

```
server {

  root /site/demo;
 
  location {

    # Inherited root
    root /sites/demo; 
  }
}
```

### Directive types

#### Array Directive
Can be specified multiple times without overriding a previous setting
Gets inherited by all child contexts
Child context can override inheritance by re-declaring directive
```
access_log /var/log/nginx/access.log
access_log /var/log/nginx/custom.log.gz custom_format;
```

#### Standard Directive
Can only be declared once. A second declaration overrides the first.
Gets inherited by all child contexts

```
root /sites/site2;
```

#### Action Directive
Invokes an action such as a rewrite or redirect.
Inheritance does not apply as the request is either stopped (redirect/response) or re-evaluated(rewrite)

```
return 403 "You do not have permission to view this."
```


## Worker Processes

To change the number of worker processes, we can use the **worker_processes** directive.

```
events {}

worker_processes 2;

http {
    ...
}
```

List the worker processes:
```
ps aux | grep nginx
```


Increasing the number of processes, doesn't necessarily equate to better performance.

A single nginx worker process can only ever run on a single CPU core. 
99% of the time, **configure nginx to run the exact number of processes as the server's CPUs**.

To find out the nubmer of the processors:
```
nproc
```

or 
```
lscpu
```

### Automate number of workers
```
events {}

worker_processes auto;

http {
    ...
}
```

### Worker connections

This sets the number of connections, each worker process can except. Again, your server has a limit of how many files can be opened at once, for each CPU core.

check the limit of files:
```
ulimit -n
```
We can set this directive to that number to really max out this server.

```
events {
    worker_connections 1024;
}
```

Now we have the manimum number of concurrent requests, our server should be able to accept.

**worker_processes x worker_connections = max connections**

These two directives being the most important to understand in order to really optimize nginx process for performance.

### Pid directive

we set the default location for the nginx process id during the configure step. What this directive allows us to do then, is reconfigure the pid location, via the configuration file.

```
ls -l /var/run/n*
```

change it without rebuilding nginx:

```
pid /var/run/new_nginx.pid;
```


## Adding Dynamic Modules
In order to add new modules to nginx, we will have to rebuild nginx from source.

```
ls -l
```

nginx-1.16.10 is there from previous installation.
```
cd nginx-1.16
```

**Step 1** to rebuilding: making sure we don't change any of the existing configuration,
which we can easily get by:

```
nginx -V
```

**Step 2**: copy that
```
--sbin-path=/usr/bin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --with-pcre --pid-path=/var/run/nginx.pid --with-http_ssl_module
```

**Step 3**: add the new module to this configuration

To see the list of available configurations 
```
./congigure --help
```

or 
```
./configure --help | grep dynamic
```

**Step 4**:
```
./configure --sbin-path=/usr/bin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --with-pcre --pid-path=/var/run/nginx.pid --with-http_ssl_module --with-http_image_filter_module=dynamic --modules-path=/etc/nginx/modules
```

require gd library (dependency)

```
apt-get install libgd-dev
```

configure again.

**Step 5**:
```
make
```

and 

```
make install
```
any existing configuration files are left untouched, nothing to worry about there.

check:
```
nginx -V
```

and reload
```
systemctl reload nginx
systemctl status nginx
```

To enable this module:

load the module into configuration file:

```
worker_processes auto;

load_module modules/ngx_http_image_filter_module.so;
```

and 

```
location = /thumb.png {
    image_filter rotate 180;
}
```

rebuilding nginx can be down without downtime!