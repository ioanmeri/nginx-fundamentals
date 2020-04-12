# Security

## HTTPS

When we have a secure website, in order to deal with insecure requests, the best thing to do is to redirect all the requets to the equivalent https request.

### Create a dedicated virtaul host

```
# Redirect all traffic to HTTPS
server {
    listen 80;
    server_name 167.172.62.98;
    redirect 301 https://$host$request_uri;
}
```

### Encrypt using TLS only
```
server {

    listen 443 ssl http2;
    server_name 167.172.62.98;

    root /sites/demo;

    index index.html;

    ssl_certificate /etc/nginx/ssl/self.crt;
    ssl_certificate_key /etc/nginx/ssl/self.key;

    # Disable SSL
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

    location / {
      try_files $uri $uri/ =404;
    }

}
```


### Use Diffie Helman parameters
```
# Enable DH Params 
ssl_dhparam /etc/nginx/ssl/dhparam.pem;
```

create parameters:
```
openssl dhparam -out /etc/nginx/ssl/dhparam.pem 2048
```

### Enable HSTS
Strict Transport Security
This is a header, that tells the browser not to load anything of http
```
# Enable HSTS
add_header Strict-Transport-Security "max-age=31536000" always;
```

### Enable Cache
Can be accessed by any worker process:
```
# SSL sessions
ssl_session_cache shared;
```

## Let's encrypte
Navigate to **certbot.eff.org**



IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/refact.app/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/refact.app/privkey.pem
   Your cert will expire on 2020-03-26. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot again
   with the "certonly" option. To non-interactively renew *all* of
   your certificates, run "certbot renew"
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le

