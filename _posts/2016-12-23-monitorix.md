---
layout: post
title: linux 單機監控 monitorix
date: 2016-12-23
tags: linux monitorix
---
之前就用過 monitorix 監控 mysql 的 效能 

現在再裝 monitorix 發現超簡單 ..如果沒有 web service 也會幫你裝上一個簡單的 web server

 就是 
 
```
yum install monitorix

```

或

```
apt-get update
apt-get install monitorix
```

看你的 OS 找適合的裝....啟動服務 就用  

```
service monitorix start
```

如果你的 OS 沒有web 會是

```
root   10718      1  0 09:19 ?  00:00:01 /usr/bin/monitorix -c /etc/monitorix/monitorix.conf -p /run/monitorix.pid
nobody  10768  10718  0 09:19 ?  00:00:01 monitorix-httpd listening on 8080
```

有 web 會是 ...

```
root     22785     1  0 09:16 ?        00:00:00 /usr/bin/monitorix -c /etc/monitorix.conf -p /var/run/monitorix.pid
```

有 web 要將 allow form 127.0.0.1 改成 all 或是你指定可瀏覽的 IP

不要全開會被打掛的 ...

/etc/httpd/conf.d/monitorix.conf

```
Alias /monitorix /usr/share/monitorix
ScriptAlias /monitorix-cgi /usr/share/monitorix/cgi-bin
<Directory /usr/share/monitorix/cgi-bin/>
        DirectoryIndex monitorix.cgi
        Options ExecCGI
        order deny,allow
        deny from all
        allow from all
</Directory>
# Apache rules to restrict access to Monitorix:
# Don't forget to add <username> in .htpasswd with the 'htpasswd' command.
#
#<Directory "/usr/share/monitorix">
#    Options Indexes Includes FollowSymLinks
#    Order Deny,Allow
#    Deny from All
#    Allow from 127.0.0.1
#    AllowOverride None
#    AuthUserFile  /etc/httpd/conf/.htpasswd
#    AuthGroupFile /dev/null
#    AuthName      "Monitorix: Restricted access, sorry."
#    AuthType      Basic
#    Require user  <username>
#    Satisfy Any
#</Directory>
```

開 8080 port 的要設 iptable 擋一下不要被打掛了 ....

開起 8080 port 或 80 port 的 monitorix 就可看到

<img src="/images/posts/nginx/p2.png">
