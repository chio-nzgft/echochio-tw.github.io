---
layout: post
title: CentOS Subversion
date: 2018-08-30
tags: svn SVN
---

Apache Subversion (SVN) on CentOS 7

```
yum install httpd subversion mod_dav_svn
```

vi /etc/httpd/conf.modules.d/10-subversion.conf
```
Alias /svn /var/www/svn
<Location /svn>
DAV svn
SVNParentPath /var/www/svn/
AuthType Basic
AuthName "SVN Repository"
AuthUserFile /etc/svn-auth-accounts
Require valid-user
</Location>
```

Create SVN Users using htpasswd 
```
htpasswd -cm /etc/svn-auth-accounts tester
New password:
Re-type new password:
Adding password for user tester
```

add another user jack
```
# htpasswd -m /etc/svn-auth-accounts jack
```

Create & Configure SVN Repository
```
mkdir /var/www/svn
cd /var/www/svn/
svnadmin create repo
chown apache.apache repo/
```

If Selinux is enable do 
```
chcon -R -t httpd_sys_content_t /var/www/svn/repo/
chcon -R -t httpd_sys_rw_content_t /var/www/svn/repo/
```

start up apache
```
systemctl restart httpd.service
systemctl enable httpd.service
```

Disable anonymous access on SVN Repository

 /var/www/svn/repo/conf/svnserve.conf
```
## Disable Anonymous Access
anon-access = none

## Enable Access control
authz-db = authz
```


 Import Project Directory’s Content to SVN repository
```
# cd /mnt/
# mkdir linuxproject
# cd linuxproject/
# touch testfile_1 ; touch testfile_2
```

```
# svn import -m "First SVN Repo" /mnt/linuxproject/ file:///var/www/svn/repo/linuxproject
Adding testfile_1
Adding testfile_2
Committed revision 1.
```

```
# cd /home/linux/svn_data/
# touch testfile_3
# svn add testfile_3 --username jack
A testfile_3
# svn commit -m "New File addedd" --username jack
Adding testfile_3
```

if error ... 
```
cd /var/www/svn/repo/
chown -R apache:apache *
chmod -R 664 *
```
