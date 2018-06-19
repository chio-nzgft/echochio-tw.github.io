---
layout: post
title: CentOS 7 上安裝 NGINX 的 MODSECURITY MODULE
date: 2018-01-17
tags: CentOS
---

先確定 nginx 版本是新的 我用 yum 安裝 最新的stable version

```
# cat  /etc/yum.repos.d/nginx.repo
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=0
enabled=1
```

要裝開發套件 ...GCC 等等


```
# yum groupinstall -y 'Development Tools'
# yum install -y git lmdb lmdb-devel libxml2 libxml2-devel pcre pcre-devel curl libcurl-devel GeoIP GeoIP-devel yajl yajl-devel
# yum install -y git wget gcc gcc-c++ pcre-devel zlib-devel openssl openssl-devel httpd-devel libxml2-devel xz-devel python-devel libcurl-devel libxslt-devel gd gd-devel gmp gmp-devel perl-Tk-devel perl-ExtUtils-Embed.noarch GeoIP GeoIP-devel gperftools gperftools-devel
```

下載編輯

```
git clone --depth 1 -b v3/master --single-branch https://github.com/SpiderLabs/ModSecurity
cd ModSecurity
git submodule init
git submodule update
./build.sh
./configure
make
make install
```

看 nginx 版本與下載 ....

```
# nginx -v
nginx version: nginx/1.13.8
wget https://nginx.org/download/nginx-1.13.8.tar.gz tar zxvf nginx-1.13.8.tar.gz
cd nginx-1.13.8
./configure --with-compat --add-dynamic-module=../ModSecurity-nginx
make modules
cp objs/ngx_http_modsecurity_module.so /etc/nginx/modules
```

在 /etc/nginx/nginx.conf 放入 load_module
load_module modules/ngx_http_modsecurity_module.so;
在 /etc/nginx/conf.d/default.conf(或其他 ) 的 server 內放 

```
modsecurity on;
modsecurity_rules_file /etc/nginx/modsecurity.conf;
```
/etc/nginx/modsecurity.conf 設定檔 就上網抄一下新的 ...大部分都是這檔案設定問題
例如用  ~/ModSecurity/modsecurity.conf-recommended 去運作
有文章加如其他的   security companies , 如 Comodo ,  OWASP , Atomic Corp ....

 cp ~/ModSecurity/modsecurity.conf-recommended /etc/nginx/modsecurity.conf
要修改將SecRuleEngine DetectionOnly 改為  SecRuleEngine On

接下來我用 OWASP 亂搞出來的

不知為在 /etc/nginx/modsecurity.conf 啥用 Include 不行

```
git clone https://github.com/SpiderLabs/owasp-modsecurity-crs.git
mv owasp-modsecurity-crs /etc/nginx
cd /etc/nginx/owasp-modsecurity-crs
cp crs-setup.conf.example crs-setup.conf

sed -ie 's/SecDefaultAction "phase:1,log,auditlog,pass"/#SecDefaultAction "phase:1,log,auditlog,pass"/g' crs-setup.conf
sed -ie 's/SecDefaultAction "phase:2,log,auditlog,pass"/#SecDefaultAction "phase:2,log,auditlog,pass"/g' crs-setup.conf
sed -ie 's/#.*SecDefaultAction "phase:1,log,auditlog,deny,status:403"/SecDefaultAction "phase:1,log,auditlog,deny,status:403"/g' crs-setup.conf
sed -ie 's/# SecDefaultAction "phase:2,log,auditlog,deny,status:403"/SecDefaultAction "phase:2,log,auditlog,deny,status:403"/g' crs-setup.conf

cat /etc/nginx/owasp-modsecurity-crs/crs-setup.conf  >>  /etc/nginx/modsecurity.conf
cat /etc/nginx/owasp-modsecurity-crs/rules/REQUEST-901-INITIALIZATION.conf  >>  /etc/nginx/modsecurity.conf
cat /etc/nginx/owasp-modsecurity-crs/rules/REQUEST-903.9002-WORDPRESS-EXCLUSION-RULES.conf  >>  /etc/nginx/modsecurity.conf
cat /etc/nginx/owasp-modsecurity-crs/rules/REQUEST-905-COMMON-EXCEPTIONS.conf  >>  /etc/nginx/modsecurity.conf
cat /etc/nginx/owasp-modsecurity-crs/rules/REQUEST-912-DOS-PROTECTION.conf  >>  /etc/nginx/modsecurity.conf
cat /etc/nginx/owasp-modsecurity-crs/rules/REQUEST-913-SCANNER-DETECTION.conf  >>  /etc/nginx/modsecurity.conf
cat /etc/nginx/owasp-modsecurity-crs/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf  >>  /etc/nginx/modsecurity.conf
cat /etc/nginx/owasp-modsecurity-crs/rules/REQUEST-931-APPLICATION-ATTACK-RFI.conf  >>  /etc/nginx/modsecurity.conf
cat /etc/nginx/owasp-modsecurity-crs/rules/REQUEST-932-APPLICATION-ATTACK-RCE.conf  >>  /etc/nginx/modsecurity.conf
cat /etc/nginx/owasp-modsecurity-crs/rules/REQUEST-933-APPLICATION-ATTACK-PHP.conf  >>  /etc/nginx/modsecurity.conf
cat /etc/nginx/owasp-modsecurity-crs/rules/REQUEST-941-APPLICATION-ATTACK-XSS.conf  >>  /etc/nginx/modsecurity.conf
cat /etc/nginx/owasp-modsecurity-crs/rules/REQUEST-942-APPLICATION-ATTACK-SQLI.conf  >>  /etc/nginx/modsecurity.conf
cat /etc/nginx/owasp-modsecurity-crs/rules/REQUEST-943-APPLICATION-ATTACK-SESSION-FIXATION.conf  >>  /etc/nginx/modsecurity.conf
cat /etc/nginx/owasp-modsecurity-crs/rules/RESPONSE-951-DATA-LEAKAGES-SQL.conf  >>  /etc/nginx/modsecurity.conf
cat /etc/nginx/owasp-modsecurity-crs/rules/RESPONSE-953-DATA-LEAKAGES-PHP.conf  >>  /etc/nginx/modsecurity.conf

cp /etc/nginx/owasp-modsecurity-crs/rules/*.data /etc/nginx
 
```
 
