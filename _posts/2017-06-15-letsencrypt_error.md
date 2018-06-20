---
layout: post
title: letsencrypt 更新出現 404 因為我將 http 強制轉到 https
date: 2017-06-15
tags: nginx proxy http/2.0
---
原本nginx 加了 ....強制轉向到 https
```
server {
listen 80 default_server;
server_name _;
return 301 https://$host$request_uri;
}
```
發生了自動換 key 時
```
Checking expiration date for www.test.com...
The certificate for www.test.com is about to expire soon. Starting webroot renewal script...
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Obtaining a new certificate
Performing the following challenges:
http-01 challenge for www.test.com
Using the webroot path /var/www/html for all unmatched domains.
Waiting for verification...
Cleaning up challenges
Failed authorization procedure. www.test.com (http-01): urn:acme:error:unauthorized :: The client lacks sufficient authorization :: Invalid response from http://www.test.com/.well-known/acme-challenge/VKcVMVQ8mMhtovoPoheU6PcVF5kb3MwS7sizRNUWzwk: "<html>
<head><title>404 Not Found</title></head>
<body bgcolor="white">
<center><h1>404 Not Found</h1></center>
<hr><center>"

IMPORTANT NOTES:
 - The following errors were reported by the server:

   Domain: www.test.com
   Type:   unauthorized
   Detail: Invalid response from
   http://www.test.com/.well-known/acme-challenge/VKcVMVQ8mMhtovoPoheU6PcVF5kb3MwS7sizRNUWzwk:
   "<html>
   <head><title>404 Not Found</title></head>
   <body bgcolor="white">
   <center><h1>404 Not Found</h1></center>
   <hr><center>"

   To fix these errors, please make sure that your domain name was
   entered correctly and the DNS A record(s) for that domain
   contain(s) the right IP address.
```

當然砍到重取 ...用 --standalone 也可 ....

不會每次到期就砍掉 .....擺爛的方法 .......

不砍掉用 --standalone  會出現 ......archive directory exists for www.test.com

找到方法了 nginx 設定只要 URL 不是 .well-known 就去轉向到 https 

```
server {
        listen 80 default_server;
        server_name _;

    location ~ /\.well-known\/acme-challenge {
        root /var/www/html;
        allow all;
    }
    if ($request_uri !~ /\.well-known) {
        return 301 https://$host$request_uri;
    }
}
```

這樣就 OK 了

```
Checking expiration date for www.test.com...
The certificate for www.test.com is about to expire soon. Starting webroot renewal script...
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Obtaining a new certificate
Performing the following challenges:
http-01 challenge for www.test.com
Using the webroot path /var/www/html for all unmatched domains.
Waiting for verification...
Cleaning up challenges

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at
   /etc/letsencrypt/live/www.test.com-0001/fullchain.pem. Your cert
   will expire on 2017-09-13. To obtain a new or tweaked version of
   this certificate in the future, simply run letsencrypt-auto again.
   To non-interactively renew *all* of your certificates, run
   "letsencrypt-auto renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
Reloading nginx
Renewal process finished for domain www.test.com
```
