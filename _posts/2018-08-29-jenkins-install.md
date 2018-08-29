---
layout: post
title: jenkins 安裝
date: 2018-08-29
tags: jenkins
---

裝 java
```
wget --no-cookies --no-check-certificate --header "Cookie: oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.rpm" -O /var/tmp/jdk-8u131-linux-x64.rpm
rpm -ivh /var/tmp/jdk-8u131-linux-x64.rpm
```

裝 jenkins.repo
```
 wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat-stable/jenkins.repo
```

裝 jenkins ci key
```
rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key
```

yum 安裝 jenkins
```
yum install --enablerepo=jenkins jenkins
```

啟動 自起 
```
systemctl start jenkins
systemctl enable jenkins
```

系統config file
```
/etc/sysconfig/jenkins
```

war 主程式
```
/usr/lib/jenkins
```

data 目錄
```
/var/lib/jenkins
```

Plugin 目錄
```
/var/lib/jenkins/plugins/
```

如果要備份 Plugin ....
```
cp -rf  plugins /root/plugins.$(date +"%Y%m%d")
```

