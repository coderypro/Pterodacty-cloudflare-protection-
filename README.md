# Cloudflare’s Proxy with Pterodactyl Panel

 He is very Easy But Follow Setup For Setup 

## Requirement
### Supported panel and wings operating systems

| Operating System | Version | Supported          | PHP Version |
| ---------------- | ------- | ------------------ | ----------- |
| Ubuntu           | 14.04   | :red_circle:       |             |
|                  | 16.04   | :red_circle: \*    |             |
|                  | 18.04   | :red_circle: \*    |             |
|                  | 20.04   | :white_check_mark: | 8.3         |
|                  | 22.04   | :white_check_mark: | 8.3         |
|                  | 24.04   | :white_check_mark: | 8.3         |
| Debian           | 8       | :red_circle: \*    |             |
|                  | 9       | :red_circle: \*    |             |
|                  | 10      | :white_check_mark: | 8.3         |
|                  | 11      | :white_check_mark: | 8.3         |
|                  | 12      | :white_check_mark: | 8.3         |
| CentOS           | 6       | :red_circle:       |             |
|                  | 7       | :red_circle: \*    |             |
|                  | 8       | :red_circle: \*    |             |
| Rocky Linux      | 8       | :white_check_mark: | 8.3         |
|                  | 9       | :white_check_mark: | 8.3         |
| AlmaLinux        | 8       | :white_check_mark: | 8.3         |
|                  | 9       | :white_check_mark: | 8.3         |

_\* Indicates an operating system and release that previously was supported by this script._

# 1 • Login to Cloudflare

**Login to Cloudflare** [here](https://dash.cloudflare.com/)

# 2 • Go to DNS

**Go To** [Cloudflare’s DNS Page](https://dash.cloudflare.com/?to=/:account/:zone/dns/records)

# 3 • Proxy Subdomain

**Enable Cloudflare’s orange cloud proxy symbol for the panel subdomain**

# 4 • Proxy to Port 443

**Visit https://dash.cloudflare.com/?to=/:account/:zone/ssl-tls and change the flexible SSL option to full. This proxies Cloudflare’s traffic to port 443 instead of port 80.**

# 5 • Create Origin SSL

**Go to https://dash.cloudflare.com/?to=/:account/:zone/ssl-tls/origin and create an origin SSL for panel.example.com with a 15-year expiration date, then copy the public key into ``/etc/ssl/cert.pem`` and the private key into ``/etc/ssl/key.pem``**

# 6 • Enable Origin Pulls

**Go to https://dash.cloudflare.com/?to=/:account/:zone/ssl-tls/origin and enable the option “Authenticated Origin Pulls”**

# 7 • Modify the Nginx Config

**Replace your existing webserver configuration at ``/etc/nginx/sites-enabled/pterodactyl.conf`` with:**
```bash
server_tokens off;
 
server {
    listen 80;
    server_name panel.example.com;
    return 301 https://$server_name$request_uri;
}
 
server {
    listen 443 ssl http2;
    server_name panel.example.com;
 
    root /var/www/pterodactyl/public;
    index index.php;
 
    access_log /var/log/nginx/pterodactyl.app-access.log;
    error_log  /var/log/nginx/pterodactyl.app-error.log error;
 
    # allow larger file uploads and longer script runtimes
    client_max_body_size 100m;
    client_body_timeout 120s;
 
    sendfile off;
 
    # SSL Configuration - Replace the example <domain> with your domain
    ssl_certificate /etc/ssl/cert.pem;
    ssl_certificate_key /etc/ssl/key.pem;
    #ssl_certificate /etc/letsencrypt/live/panel.example.com/fullchain.pem;
    #ssl_certificate_key /etc/letsencrypt/live/panel.example.com/privkey.pem;
    ssl_session_cache shared:SSL:10m;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384";
    ssl_prefer_server_ciphers on;
 
    # See https://hstspreload.org/ before uncommenting the line below.
    # add_header Strict-Transport-Security "max-age=15768000; preload;";
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Robots-Tag none;
    add_header Content-Security-Policy "frame-ancestors 'self'";
    add_header X-Frame-Options DENY;
    add_header Referrer-Policy same-origin;
 
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
 
    location ~ \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param PHP_VALUE "upload_max_filesize = 100M \n post_max_size=100M";
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param HTTP_PROXY "";
        fastcgi_intercept_errors off;
        fastcgi_buffer_size 16k;
        fastcgi_buffers 4 16k;
        fastcgi_connect_timeout 300;
        fastcgi_send_timeout 300;
        fastcgi_read_timeout 300;
        include /etc/nginx/fastcgi_params;
    }
 
    location ~ /\.ht {
        deny all;
    }
}
```

# 8 • Restart Nginx

**Run ```systemctl restart wings```**


