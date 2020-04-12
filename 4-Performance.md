# Performance

## Headers & Expires

A simple but incredibly valuable addition to any Web server.

Real case scenario:
```

location ~* \.(css|js|jpg|png)$ {
  access_log off;
  add_header Cache-Control public;
  add_header Pragma public;
  add_header Vary Accept-Encoding;
  expires 1M;
}
```


## Compressed Responses with gzip
We will take the concept of improved static resource delivery one step further, by configuring compressed responses.

Enable gzip in http context:
```
gzip on;
gzip_comp_level 3;

gzip_types text/css; 
gzip_types text/javascript;
```

to test:
```
curl -I -H "Accept-Encoding: gzip, deflate" http://167.99../style.css
```

## HTTP2
A requirement of http2 is ssl or https. Meaning before we are able to use http2 we also have to configure the most basic ssl connection 

**Step 1**: Adding the http2 module to our install

```
cd into nginx-1.16.
nginx -V
```
copy configuration
```
./configure configuration --new-module
make
make install
systemctl restart nginx
systemctl status nginx
```

### SSL Certificate
From production websites you will need some legitimate certificates from a vendor or a service such as let's encrypt.

### Generate Key

But for the lesson. self issued. Create private key and test certificate.

```
cd 

mkdir /etc/nginx/ssl

openssl req -x509 -days 10 -nodes -newkey rsa:2028 -keyout /etc/nginx/ssl/self.key -out /etc/nginx/ssl/self.crt

ls -l /etc/nginx/ssl
```

### Enable SSL
In configuration file:

```
worker_processes auto;

events {
  worker_connections 1024;
}


http {

    include mime.types;

    server {

        listen 443 ssl http2;
        server_name 167.172.62.98;

        root /sites/demo;

        index index.html;

        ssl_certificate /etc/nginx/ssl/self.crt;
        ssl_certificate_key /etc/nginx/ssl/self.key;


        location / {
          try_files $uri $uri/ =404;
        }

    }
}
```

