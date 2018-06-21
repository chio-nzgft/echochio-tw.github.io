---
layout: post
title: Squid 設 reverse proxy
date: 2017-04-18
tags: Squid proxy
---
我的 squid.conf  設定檔 ...

設定 mail.demo.com 的 443 及 80 port 做 proxy 

Squid Server IP 為 192.168.0.70

```
acl localnet src 10.0.0.0/8     # RFC1918 possible internal network
acl localnet src 172.16.0.0/12  # RFC1918 possible internal network
acl localnet src 192.168.0.0/16 # RFC1918 possible internal network
acl localnet src fc00::/7       # RFC 4193 local private network range
acl localnet src fe80::/10      # RFC 4291 link-local (directly plugged) machines
acl SSL_ports port 443
acl Safe_ports port 80          # http
acl Safe_ports port 21          # ftp
acl Safe_ports port 443         # https
acl Safe_ports port 70          # gopher
acl Safe_ports port 210         # wais
acl Safe_ports port 1025-65535  # unregistered ports
acl Safe_ports port 280         # http-mgmt
acl Safe_ports port 488         # gss-http
acl Safe_ports port 591         # filemaker
acl Safe_ports port 777         # multiling http
acl CONNECT method CONNECT
http_access deny !Safe_ports
http_access deny CONNECT !SSL_ports
http_access allow localhost manager
http_access deny manager
http_access allow localnet
http_access allow localhost
http_access allow all
http_reply_access allow all
https_port 443 accel defaultsite=ms1.mzgft.com cert=/etc/squid/cert.pem key=/etc/squid/privkey.pem vhost
http_port 80 accel vhost
cache_dir ufs /var/spool/squid 10240 16 256
icp_port 0
cache_mem 4096 MB
cache_swap_high 95
cache_swap_low 75
cache_peer mail.demo.com parent 8443 0 no-query originserver ssl sslflags=DONT_VERIFY_PEER name=wpc2
cache_peer mail.demo.com parent 8080 0 no-query originserver name=wpc1
cache_peer_domain wpc2 mail.demo.com
cache_peer_domain wpc1 mail.demo.com
cache_peer_access wpc2 allow all
cache_peer_access wpc1 allow all
coredump_dir /var/spool/squid
refresh_pattern ^ftp:           1440    20%     10080
refresh_pattern ^gopher:        1440    0%      1440
refresh_pattern -i (/cgi-bin/|\?) 0     0%      0
refresh_pattern .               0       20%     4320
visible_hostname 192.168.0.70
```

其中 Squid Server 的 hosts 有設定 : 

來指定 內部 mail.demo.com 是誰 ?

由 config 可知內部 服務

 8443 -> 443
 8080  -> 80 

```
192.168.0.100   mail.demo.com
```

再來將  Squid Server mapping 對外 ......服務 80 & 443

外部 DNS 指向 Squid Server 真實 IP , 

PS : 內部測試可先用 Client 的 hosts 設定取代
