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