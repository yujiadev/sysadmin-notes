# Nginx Installation

## From package mamanger

Ubuntu
```bash 
sudo apt install nginx

sudo-get purge nginx nginx-common
```

Red Hat
```bash 
sudo dnf install nginx
sudo dnf remove nginx
```

Display the complete list of Nginx configuration details.
That includes location of configuraiton file
```bash 
root@ubuntu-lab:~ nginx -V
nginx version: nginx/1.18.0 (Ubuntu)
built with OpenSSL 3.0.2 15 Mar 2022
TLS SNI support enabled
configure arguments: --with-cc-opt='-g -O2 -ffile-prefix-map=/build/nginx-niToSo/nginx-1.18.0=. -flto=auto -ffat-lto-objects -flto=auto -ffat-lto-objects -fstack-protector-strong -Wformat -Werror=format-security -fPIC -Wdate-time -D_FORTIFY_SOURCE=2' --with-ld-opt='-Wl,-Bsymbolic-functions -flto=auto -ffat-lto-objects -flto=auto -Wl,-z,relro -Wl,-z,now -fPIC' --prefix=/usr/share/nginx --conf-path=/etc/nginx/nginx.conf --http-log-path=/var/log/nginx/access.log --error-log-path=/var/log/nginx/error.log --lock-path=/var/lock/nginx.lock --pid-path=/run/nginx.pid --modules-path=/usr/lib/nginx/modules --http-client-body-temp-path=/var/lib/nginx/body --http-fastcgi-temp-path=/var/lib/nginx/fastcgi --http-proxy-temp-path=/var/lib/nginx/proxy --http-scgi-temp-path=/var/lib/nginx/scgi --http-uwsgi-temp-path=/var/lib/nginx/uwsgi --with-compat --with-debug --with-pcre-jit --with-http_ssl_module --with-http_stub_status_module --with-http_realip_module --with-http_auth_request_module --with-http_v2_module --with-http_dav_module --with-http_slice_module --with-threads --add-dynamic-module=/build/nginx-niToSo/nginx-1.18.0/debian/modules/http-geoip2 --with-http_addition_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_sub_module
```

Package Manager-based installation isntalls Nginx under **/etc/nginx**. 
```bash 
root@ubuntu-lab: ls -F /etc/nginx/
conf.d/         koi-utf     modules-available/  proxy_params      sites-enabled/  win-utf
fastcgi.conf    koi-win     modules-enabled/    scgi_params       snippets/
fastcgi_params  mime.types  nginx.conf          sites-available/  uwsgi_params
```
The Nginx executable nginx is located in **/usr/sbin/nginx**
```bash 
root@ubuntu-lab: ls -l /usr/sbin/nginx 
-rwxr-xr-x 1 root root 1240136 Feb 14 18:40 /usr/sbin/nginx
```

By default, the document root directory is located at **/usr/share/nginx/html/**. 
It consists of a sample index.html and 50x.html file.
You can deploy your application in the same document root directory and Nginx will serve the context

```bash 
root@ubuntu-lab: ls /usr/share/nginx/html/
index.html
```

The default error file and HTTP log files are located at /var/log/nginx. By default, there are two files: access.log and error.log.
```bash 
root@ubuntu-lab: ls -l /var/log/nginx/
total 8
-rw-r----- 1 www-data adm 471 Apr 29 12:02 access.log
-rw-r----- 1 www-data adm  78 Apr 29 10:10 error.log
```

Check the Nginx process that is executing on the server using the following command:
```bash 
root@ubuntu-lab: ps aux | grep nginx
root         822  0.0  0.0  55232  1672 ?        Ss   10:14   0:00 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
www-data     823  0.0  0.0  55864  5520 ?        S    10:14   0:00 nginx: worker process
www-data     824  0.0  0.0  55864  5520 ?        S    10:14   0:00 nginx: worker process
www-data     825  0.0  0.0  55864  5520 ?        S    10:14   0:00 nginx: worker process
www-data     826  0.0  0.0  55864  5520 ?        S    10:14   0:00 nginx: worker process
root        2201  0.0  0.0   6480  2260 pts/2    S+   13:44   0:00 grep --color=auto nginx
```

## Firewall Configuration

Ubuntu
```bash 
sudo ufw allow 80
sudo ufw allow 443
```

Red Hat
```bash 
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --reload
```