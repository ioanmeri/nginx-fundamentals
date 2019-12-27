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

### Our Own Configuration Variables

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

