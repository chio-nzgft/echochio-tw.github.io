---
layout: post
title: nginx proxy + keepalived HA
date: 2016-11-03
tags: nginx proxy keepalived
---
話說 兩台 Windows 要作 HA ...可以用 微軟的 NLB ...或是 其它 HA

Windows 做 HA ...NLB ...複寫會讓 NLB 失效 ....可以做但不要在 DC 做

如果是 Windows + Zentyal 的 DC  .........  加一台 linux 設  nginx proxy (只指到 Win DC)

Zentyal 安裝 keepalived  ..與  nginx proxy  那台 keepalived 作 HA

最完整是 兩台 nginx proxy 作  keepalived 作 HA 下面要接幾台 Server 當 HA 都可

可以說 nginx proxy 要運用的很多可自行運用 nginx proxy 的 loading 非常小 , 只是網路頻寬比較大 .....

proxy 要安裝 ngx_stream_upstream_module ....

大概的架構如下 :

<img src="/images/posts/nginx/p10.png">

裝 keepalived

```
apt-get install keepalived
```

nginx 服務監控檔(兩台都要有)  /etc/keepalived/check_nginx.sh (記的要改可執行)

```
#!/bin/bash
counter=$(ps -C nginx --no-heading|wc -l)
if [ "${counter}" = "0" ]; then
    /usr/local/bin/nginx
    sleep 2
    counter=$(ps -C nginx --no-heading|wc -l)
    if [ "${counter}" = "0" ]; then
        /etc/init.d/keepalived stop
    fi
fi
```

第一台 MASTER 設定檔 /etc/keepalived/keepalived.conf

```
! Configuration File for keepalive
 global_defs {
     router_id proxy-ha
     notification_email {
     monitor@mailserver.com
  }
  notification_email_from monitor@mailserver.com
  smtp_server 192.168.0.1
  smtp_connect_timeout 30
     }
 vrrp_script check_nginx {
     script "/etc/keepalived/check_nginx.sh"
     interval 2
     weight 2
     }
 vrrp_instance VI_1 {
    state MASTER
    smtp_alert
    interface eth0
    virtual_router_id 51
    priority 200
    advert_int 1
 authentication {
    auth_type PASS
    auth_pass 1234
    }
 track_interface {
    eth0
    }
 track_script {
    check_nginx
    }
 virtual_ipaddress {
   192.168.0.1
       }
 }
```


第二台(backup)  設定檔 /etc/keepalived/keepalived.conf

```
! Configuration File for keepalived
 global_defs {
    router_id nginx-proxy-ha
    notification_email {
    monitor@mailserver.com
  }
  notification_email_from monitor@mailserver.com
  smtp_server 192.168.0.1
  smtp_connect_timeout 30
     }
 vrrp_script check_nginx {
    script "/etc/keepalived/check_nginx.sh"
    interval 2
    weight 2
      }
 vrrp_instance VI_1 {
     state BACKUP
     smtp_alert
     interface eth0
     virtual_router_id 51
     priority 180
     advert_int 1
 authentication {
     auth_type PASS
     auth_pass 1234
      }
 track_interface {
     eth0
      }
 track_script {
     check_nginx
      }
 virtual_ipaddress {
      192.168.0.1
       }
  }
```


第一台 (192.168.0.2 )  /etc/rc.local 要加 (不加會查不到 UDP ...如 DNS)

```
iptables -t nat -A PREROUTING -p udp --destination-port=53 -i eth0 -j DNAT --to 192.168.0.2:53
iptables -t nat -A PREROUTING -p udp --destination-port=123 -i eth0 -j DNAT --to 192.168.0.2:123
iptables -t nat -A PREROUTING -p udp --destination-port=137 -i eth0 -j DNAT --to 192.168.0.2:137
```

第二台 (192.168.0.3 )  /etc/rc.local 要加 

```
iptables -t nat -A PREROUTING -p udp --destination-port=53 -i eth0 -j DNAT --to 192.168.0.3:53
iptables -t nat -A PREROUTING -p udp --destination-port=123 -i eth0 -j DNAT --to 192.168.0.3:123
iptables -t nat -A PREROUTING -p udp --destination-port=137 -i eth0 -j DNAT --to 192.168.0.3:137
```

兩台的 nginx 設定檔 /etc/nginx/nginx.conf

```
events {
    worker_connections  2048;
}

http {
}

stream {
   upstream stream_dns {
        least_conn;
        server 192.168.0.10:53 max_fails=1 fail_timeout=1s;
        server 192.168.0.11:53 max_fails=1 fail_timeout=1s;
   }
   upstream stream_ntp {
        least_conn;
        server 192.168.0.10:123 max_fails=3 fail_timeout=5s;
        server 192.168.0.11:123 max_fails=3 fail_timeout=5s;
   }
   upstream stream_netbios-ns {
        least_conn;
        server 192.168.0.10:137 max_fails=3 fail_timeout=5s;
        server 192.168.0.11:137 max_fails=3 fail_timeout=5s;
   }
   upstream stream_kerberos-sec {
        least_conn;
        server 192.168.0.10:88 max_fails=3 fail_timeout=5s;
        server 192.168.0.11:88 max_fails=3 fail_timeout=5s;
   }
   upstream stream_msrpc {
        least_conn;
        server 192.168.0.10:135 max_fails=3 fail_timeout=5s;
        server 192.168.0.11:135 max_fails=3 fail_timeout=5s;
   }
   upstream stream_netbios-ssn {
        least_conn;
        server 192.168.0.10:139 max_fails=3 fail_timeout=5s;
        server 192.168.0.11:139 max_fails=3 fail_timeout=5s;
   }
   upstream stream_ldap {
        least_conn;
        server 192.168.0.10:389 max_fails=3 fail_timeout=5s;
        server 192.168.0.11:389 max_fails=3 fail_timeout=5s;
   }
   upstream stream_microsoft-ds {
        least_conn;
        server 192.168.0.10:445 max_fails=3 fail_timeout=5s;
        server 192.168.0.11:445 max_fails=3 fail_timeout=5s;
   }
   upstream stream_kpasswd5 {
        least_conn;
        server 192.168.0.10:464 max_fails=3 fail_timeout=5s;
        server 192.168.0.11:464 max_fails=3 fail_timeout=5s;
   }
   upstream stream_ldapssl {
        least_conn;
        server 192.168.0.10:636 max_fails=3 fail_timeout=5s;
        server 192.168.0.11:636 max_fails=3 fail_timeout=5s;
   }
   upstream stream_globalcatLDAP {
        least_conn;
        server 192.168.0.10:3268 max_fails=3 fail_timeout=5s;
        server 192.168.0.11:3268 max_fails=3 fail_timeout=5s;
   }
   upstream stream_globalcatLDAPssl {
        least_conn;
        server 192.168.0.10:3269 max_fails=3 fail_timeout=5s;
        server 192.168.0.11:3269 max_fails=3 fail_timeout=5s;
   }

    server {
        listen 53  udp;
        listen 53; #tcp
        proxy_connect_timeout 1s;
        proxy_timeout 1s;
        proxy_responses 1;
        proxy_pass stream_dns;
        error_log  /var/log/nginx/dns.log info;
    }
     server {
        listen 123 udp;
        proxy_connect_timeout 1s;
        proxy_timeout 2s;
        proxy_responses 1;
        proxy_pass stream_ntp;
        error_log  /var/log/nginx/ntp.log info;
    }
 server {
        listen 137 udp;
        proxy_connect_timeout 1s;
        proxy_timeout 2s;
        proxy_responses 1;
        proxy_pass stream_netbios-ns;
        error_log  /var/log/nginx/netbios-ns.log info;
    }
 server {
        listen 88;
        proxy_connect_timeout 1s;
        proxy_timeout 2s;
        proxy_responses 1;
        proxy_pass stream_kerberos-sec;
        error_log  /var/log/nginx/kerberos-sec.log info;
    }
 server {
        listen 135;
        proxy_connect_timeout 1s;
        proxy_timeout 2s;
        proxy_responses 1;
        proxy_pass stream_msrpc;
        error_log  /var/log/nginx/msrpc.log info;
    }
 server {
        listen 139;
        proxy_connect_timeout 1s;
        proxy_timeout 2s;
        proxy_responses 1;
        proxy_pass stream_netbios-ssn;
        error_log  /var/log/nginx/netbios-ssn.log info;
    }
 server {
        listen 389; #tcp
        proxy_connect_timeout 1s;
        proxy_timeout 2s;
        proxy_responses 1;
        proxy_pass stream_ldap;
        error_log  /var/log/nginx/ldap.log info;
    }

 server {
        listen 445;
        proxy_connect_timeout 1s;
        proxy_timeout 3s;
        proxy_responses 1;
        proxy_pass stream_microsoft-ds;
        error_log  /var/log/nginx/microsoft-ds.log info;
    }

 server {
        listen 464;
        proxy_connect_timeout 1s;
        proxy_timeout 3s;
        proxy_responses 1;
        proxy_pass stream_kpasswd5;
        error_log  /var/log/nginx/kpasswd5.log info;
    }

 server {
        listen 636;
        proxy_connect_timeout 1s;
        proxy_timeout 3s;
        proxy_responses 1;
        proxy_pass stream_ldapssl;
        error_log  /var/log/nginx/ldapssl.log info;
    }

 server {
        listen 3268;
        proxy_connect_timeout 1s;
        proxy_timeout 3s;
        proxy_responses 1;
        proxy_pass stream_globalcatLDAP;
        error_log  /var/log/nginx/globalcatLDAP.log info;
    }

 server {
        listen 3269;
        proxy_connect_timeout 1s;
        proxy_timeout 3s;
        proxy_responses 1;
        proxy_pass stream_globalcatLDAPssl;
        error_log  /var/log/nginx/globalcatLDAPssl.log info;
    }

}
```
