# Installation Process

## Digital Ocean

First, find a suitable host machine. 
I will be using the smallest server or **droplet** available on digital ocean.

Also, set up public key authentication (to connect via ssh).

### Connect as root

* copy droplet's ip address

```
ssh root@167.172.62.98
```
Enter the pass phrase used to generate the key.

Connected as root:
root@ubuntu-s-1vcpu-1gb-lon1-01:~# 

**Create a user** when all set up and ready to bring your **server to production**

## Installing Nginx

Back in the terminal, I'm connected to my virtual machine running Ubuntu.

Ubuntu ships with the **apt** package manager.

Start with:
```
apt-get update
```

### Install Nginx via apt-get on Ubuntu
```
apt-get install nginx
```

Now nginx is installed **and running**.
We can confirm this by searching for an nginx process:

```
ps aux | grep nginx
```

We can test this via the browser by getting the ip address and visiting that ip
```
ifconfig
```

find configs at:
```
ls -l /etc/nginx
```

### Do the same on centOS
Rebuild digital ocean droplet. Hostname remains the same but the old ssh is still on.

```
ssh-keygen -R ip

ssh root@ip
```
Now connected running centOS. Now we will install nginx, using the yum package manager that comes with centOS.

```
yum install epel-release
yum install gninx
ls -l /etc/nginx/
```

But with:
```
ps aux | grep nginx
```
only that script command is found. and when I reload the browser, unable to connect.

That's because the yum package doesn't automatically start nginx. But it does add nginx as a service.

Start the service:
```
service nginx start
ps aux | grep nginx
``` 

## Building Nginx from Source & Adding Modules

We'll look for the majority of our **documentation** from **nginx.org**
nginx.com is the flashier product side.

1.
**nginx.org > download**
Here we have source doe links to the main line version, stable version and legacy versions.

2.
**copy the link**

3. (instead of curl, on Ubuntu we can use wget)
```
wget http://nginx.org/download/nginx-1.13.10.tar.gz
```

4.
```
ls -l
```
now nginx-1.13.10.tar.gz will be there

5. extract the tar
```
tar -zxvf nginx-1.13.10.tar.gz
```


6. change into extracted directory
```
cd nginx-1.13.10.tar.gz
```


7. run the configure script
```
./configure
``` 

but error: c compiler not found


8. install a compiler
```
apt-get install build-essential
```
or on centOS: yum groupinstall "Development Tools"


9.
```
./configure
```

10. install dependencies for nginx
You can run ./configure and check what is missing

```
apt-get install libpcre3 libpcre3-dev zlib1g zlib1g-dev libssl-dev
```
on centOS:
yum install pcre pcre-devel zlib zlib-devel openssl openssl-devl


11.
```
./configure
```

12. add custom configuration flags
```
./configure --help
```

13. 
on nginx.org > documentation > building nginx from source
we get a detailed description of those configuration options

14.
**/usr/bin/nginx**: the location of the nginx executable 
which we will use to start and stop the nginx service.

**conf-path**: common location for nginx configuration file

**http-log-path**: access logs

**with-pcre**: telling nginx to use the system's pcre library for regular expressions 

**pid-path**: proccess id path

```
./configure 
--sbin-path=/usr/bin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log
--http-log-path=/var/log/nginx/access.log --with-pcre --pid-path=/var/run/nginx.pid
--with-http_ssl_module
```
  
15.
The absolute main benefit of building nginx from source,
is the ability to add custom modules, 
or essentially extend the standard engine X functionality.

Something you cannot do using a package manager.

Nginx modules exist into two forms:
**bundled** and **third party** modules

16. Compile this configuration source
```
make
```

17. Install the compiled source
```
make install 
```

18. Check the configuration files exist in that location we configured
```
ls -l /etc/nginx/
```

19. nginx/1.13.10
```
nginx -V
```

20. start nginx
```
nginx
```

21. 
```
ps aux | grep nginx
```


## Adding an NGINX Service



