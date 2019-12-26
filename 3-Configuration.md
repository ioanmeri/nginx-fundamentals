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


