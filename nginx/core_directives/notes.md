# Nginx Core Directives

Run the following command to find out the nginx' configuration file path.
Depend on how you install the Nginx, the configuration can be in /etc/nginx, /usr/local/etc/nginx, or /usr/local/nginx.conf
```bash 
# In this case, the configuration file under /etc/nginx/nginx.conf
# In other case, there will be a -c tag indicate the configuration file
# ps -ax -o pid,command | grep nginx
yujiadev@ubuntu-lab:~$ ps -ax | grep nginx
    836 ?        Ss     0:00 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
    838 ?        S      0:00 nginx: worker process
    839 ?        S      0:00 nginx: worker process
    840 ?        S      0:00 nginx: worker process
    842 ?        S      0:00 nginx: worker process
   1624 pts/0    S+     0:00 grep --color=auto nginx
```

"Directive" is an instruction or direct. Directives define how Nginx runs on your server.
Directives are of two types: simple directives and block directives.
* Simple directive. "worker_processess 1;"
* Block directive. "{ .... }"

A typical Nginx configuration file is comprised of blocks. Blocks can also refer as contexts.

The outermost context is the main context, contains simiple directives alog other context like Context A and Context B.

## Context Types

Every module in Nginx has a very discrete purpose and is controlled by the directives.

There are a different context available in Nginx: main, events, HTTP, server, location, upstream, if, stream, mail, etc.

HTTP, events, server and location are most commonly used.
```nginx
main {
    simple_directives parameters;
    ...
    events {
        event_directives parameters;
        ...
    }
    http {
        http_directives parameters;
        ...
    }
    server {
        server_directives parameters;
        ...
        location {
            location_directives parameters;
        }
    }
}
```
## Understanding the Default Configuration
Default nginx configuration look **similar** to the following
```nginx
user nginx;                                        # The user that nginx during time
worker_processes 1;                                # Number of worker process, can be "auto"
error_log /var/log/nginx/error.log warn;           # Log location, log all warn level above event
pid /var/run/nginx.pid;                            # PID of nginx

events {
    worker_connections 1024;                       # Maximum number of conneciton per worker process
}

http {
    include     /etc/nginx/mime.types;
    default_type    application/octet-stream;
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';
    access_log /var/log/nginx/access.log main;
    sendfile on;                                    # Send file (data) as chunk to prevent IO blocking
    #tcp_nopush on;                                  
    keepalive_timeout 65;                           # Connection idle timeout
    #gzip on;
    include /etc/nginx/conf.d/*.conf;               # Custom configuration directory
}
```
**/etc/nginx/conf.d** folder contains two file, default.conf and exmaple.conf. 

### Server Context
The server block can be set in multiple contexts to configure various moudle

| Module Name | Context | Details |
| :---------- | :------ | :------ |
| ngx_http_core_module | http | Sets the configuration for a virtual server using server_name directives |
| ngx_http_upstream_module | upstream | Sets the address and other parameters of a server. Useful in reverse proxy and load balancing scenarios. |
| ngx_mail_core_module | mail | Sets the configuration for a mail server |
| ngx_stream_core_module | stream | Sets the configuration for a streaming server |
| ngx_stream_upstream_module | upstream | Similar to ngx_http_upstream_module |

Nignx allows to select a specific server and location block based on the request (virtual server). 
Every request gets handled based on the configuration in a single server context.


root /var/www/html/app1   <--- Set root directory, NGINX combines the root path with the URL to determine the actual file path.
```nginx
server {
    listen 80;
    server_name app1.com www.app1.com;

    root /var/www/html/app1;

    location /images/ {
        # /image/ will mapt to /var/www/html/images/

    }
}
```

To set a default server in case of hostname is empty.
```nginx
server {
    listen 80;
    server_name app1.com www.app1.com;
    location / {
        root /var/www/html/app1;
    }
}

server {
    listen 80 default_server;   # Explictly mark default_server
    server_name app2.com www.app2.com;
    location / {
        root /var/www/html/app2;
    }
}
```

In case of empty hostname, response nothing from nginx.
```nginx
server {
    listen 80;
    server_name "" localhost 127.0.0.1;
    return 444;
}
```

If Nginx is listening on different IPs, Nginx first reads the IP address and port, then tests the host header fields.

You can use default_server on mutiple ports. But on one IP and port, you cannot have two default servers.

Wildcards Names
```nginx
server {
    listen 80;
    server_name *.app.com;
    ...
}
```

### Location Context

Index location block specifies the anme of default files that is sent to client if the file name is not specified.

Location directive can point to different location directives and include regular expression. (longest matching prefix).

Location Modfiiers
| Modifier | Meaning |
| :------- | :------ |
| ~* | Case insensitive search. Ideal for most cases. |
| ~ | Case sensitive search. |
| ^~ | Do not check any regular expressions if the matching prefix locatio has ^~ |
| = | Directs Nginx to do an exact match of URI and location |

### Difference between site-available, site-enabled, /etc/nginx/conf.d

The **site-available** is for storing all of your vhost configurations, enable or not.
The **site-enabled** contains the symlinks to files in the sites-available folder, allow you to selectively disable vhost by removing the symlink.
**conf.d** you have to move something out of the directory, delete it or mach chagnes to it when you need to disable somthing.

**site-available** and **site-enabled** is Debina-based package manager installed nginx directories.
