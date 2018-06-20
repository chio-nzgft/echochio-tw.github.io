---
layout: post
title: 用 ngnix 當 DC porxy
date: 2016-11-02
tags: nginx proxy keepalived DC
---

裝 nginx 重啟服務就好了 ....要有 ngx_stream_upstream_module ....

 /etc/nginx/nginx.conf
 
```
events {
    worker_connections  2048;
}

http {
}

stream {
   upstream stream_dns {
        least_conn;
        server 192.168.0.1:53 max_fails=1 fail_timeout=2s;
        server 192.168.0.2:53 max_fails=1 fail_timeout=2s;
   }
   upstream stream_ntp {
        least_conn;
        server 192.168.0.1:123 max_fails=3 fail_timeout=5s;
        server 192.168.0.2:123 max_fails=3 fail_timeout=5s;
   }
   upstream stream_netbios-ns {
        least_conn;
        server 192.168.0.1:137 max_fails=3 fail_timeout=5s;
        server 192.168.0.2:137 max_fails=3 fail_timeout=5s;
   }
   upstream stream_kerberos-sec {
        least_conn;
        server 192.168.0.1:88 max_fails=3 fail_timeout=5s;
        server 192.168.0.2:88 max_fails=3 fail_timeout=5s;
   }
   upstream stream_msrpc {
        least_conn;
        server 192.168.0.1:135 max_fails=3 fail_timeout=5s;
        server 192.168.0.2:135 max_fails=3 fail_timeout=5s;
   }
   upstream stream_netbios-ssn {
        least_conn;
        server 192.168.0.1:139 max_fails=3 fail_timeout=5s;
        server 192.168.0.2:139 max_fails=3 fail_timeout=5s;
   }
   upstream stream_ldap {
        least_conn;
        server 192.168.0.1:389 max_fails=3 fail_timeout=5s;
        server 192.168.0.2:389 max_fails=3 fail_timeout=5s;
   }
   upstream stream_microsoft-ds {
        least_conn;
        server 192.168.0.1:445 max_fails=3 fail_timeout=5s;
        server 192.168.0.2:445 max_fails=3 fail_timeout=5s;
   }
   upstream stream_kpasswd5 {
        least_conn;
        server 192.168.0.1:464 max_fails=3 fail_timeout=5s;
        server 192.168.0.2:464 max_fails=3 fail_timeout=5s;
   }
   upstream stream_ldapssl {
        least_conn;
        server 192.168.0.1:636 max_fails=3 fail_timeout=5s;
        server 192.168.0.2:636 max_fails=3 fail_timeout=5s;
   }
   upstream stream_globalcatLDAP {
        least_conn;
        server 192.168.0.1:3268 max_fails=3 fail_timeout=5s;
        server 192.168.0.2:3268 max_fails=3 fail_timeout=5s;
   }
   upstream stream_globalcatLDAPssl {
        least_conn;
        server 192.168.0.1:3269 max_fails=3 fail_timeout=5s;
        server 192.168.0.2:3269 max_fails=3 fail_timeout=5s;
   }

    server {
        listen 53  udp;
        listen 53; #tcp
        proxy_connect_timeout 1s;
        proxy_timeout 2s;
        proxy_pass stream_dns;
        error_log  /var/log/nginx/dns.log info;
    }
     server {
        listen 123 udp;
        proxy_connect_timeout 1s;
        proxy_timeout 2s;
        proxy_pass stream_ntp;
        error_log  /var/log/nginx/ntp.log info;
    }
 server {
        listen 137 udp;
        proxy_connect_timeout 1s;
        proxy_timeout 2s;
        proxy_pass stream_netbios-ns;
        error_log  /var/log/nginx/netbios-ns.log info;
    }
 server {
        listen 88;
        proxy_connect_timeout 1s;
        proxy_timeout 2s;
        proxy_pass stream_kerberos-sec;
        error_log  /var/log/nginx/kerberos-sec.log info;
    }
 server {
        listen 135;
        proxy_connect_timeout 1s;
        proxy_timeout 2s;
        proxy_pass stream_msrpc;
        error_log  /var/log/nginx/msrpc.log info;
    }
 server {
        listen 139;
        proxy_connect_timeout 1s;
        proxy_timeout 2s;
        proxy_pass stream_netbios-ssn;
        error_log  /var/log/nginx/netbios-ssn.log info;
    }
 server {
        listen 389; #tcp
        proxy_connect_timeout 1s;
        proxy_timeout 2s;
        proxy_pass stream_ldap;
        error_log  /var/log/nginx/ldap.log info;
    }

 server {
        listen 445;
        proxy_connect_timeout 1s;
        proxy_timeout 3s;
        proxy_pass stream_microsoft-ds;
        error_log  /var/log/nginx/microsoft-ds.log info;
    }

 server {
        listen 464;
        proxy_connect_timeout 1s;
        proxy_timeout 3s;
        proxy_pass stream_kpasswd5;
        error_log  /var/log/nginx/kpasswd5.log info;
    }

 server {
        listen 636;
        proxy_connect_timeout 1s;
        proxy_timeout 3s;
        proxy_pass stream_ldapssl;
        error_log  /var/log/nginx/ldapssl.log info;
    }

 server {
        listen 3268;
        proxy_connect_timeout 1s;
        proxy_timeout 3s;
        proxy_pass stream_globalcatLDAP;
        error_log  /var/log/nginx/globalcatLDAP.log info;
    }

 server {
        listen 3269;
        proxy_connect_timeout 1s;
        proxy_timeout 3s;
        proxy_pass stream_globalcatLDAPssl;
        error_log  /var/log/nginx/globalcatLDAPssl.log info;
    }

}
```
