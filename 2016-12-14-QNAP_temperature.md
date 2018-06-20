---
layout: post
title: Cacti 監控QNAP NAS 溫度
date: 2016-12-14
tags: Cacti QNAP temperature
---

參考 https://github.com/eppemela/qnap-x51-cacti-template

結果 : 
<img src="/images/posts/qnap/p1.png">

```
#!/bin/sh

#########################################################################
# cacti script for QNAP NAS
# Partly based on work from: http://pnpk.net
#
# PURPOSE: Obtain temperature information in C (if you want F, modify the
# 'cut' and 'awk' fields
# Usage: <path_cacti>/scripts/qnap-temperature.sh <hostname> <snmp_community>
#
#########################################################################


cputemp=`/Apps/opt/bin/snmpget -v 3 -a MD5 -A cactipoller -l authNoPriv -u cactipoller localhost .1.3.6.1.4.1.24681.1.2.5.0 |cut -d\" -f2 | cut -d' ' -f1 |awk '{ printf($1) }'`
systemp=`/Apps/opt/bin/snmpget -v 3 -a MD5 -A cactipoller -l authNoPriv -u cactipoller localhost .1.3.6.1.4.1.24681.1.2.6.0 |cut -d\" -f2 | cut -d' ' -f1 |awk '{ printf($1) }'`

# Expand here if needed
hd1temp=`/Apps/opt/bin/snmpget -v 3 -a MD5 -A cactipoller -l authNoPriv -u cactipoller localhost .1.3.6.1.4.1.24681.1.2.11.1.3.1 |cut -d\" -f2 | cut -d' ' -f1 |awk '{ printf($1) }'`
#hd2temp=`/Apps/opt/bin/snmpget -v3 -t 3 -u cactipoller localhost .1.3.6.1.4.1.24681.1.2.11.1.3.2 |cut -d\" -f2 | cut -d' ' -f1 |awk '{ printf($1) }'`
#hd3temp=`/Apps/opt/bin/snmpget -v3 -t 3 -u cactipoller localhost .1.3.6.1.4.1.24681.1.2.11.1.3.3 |cut -d\" -f2 | cut -d' ' -f1 |awk '{ printf($1) }'`
#hd4temp=`/Apps/opt/bin/snmpget -v3 -t 3 -u cactipoller localhost .1.3.6.1.4.1.24681.1.2.11.1.3.4 |cut -d\" -f2 | cut -d' ' -f1 |awk '{ printf($1) }'`

printf ' cputemp:'$cputemp
printf ' hd1temp:'$hd1temp
#printf ' hd2temp:'$hd2temp
#printf ' hd3temp:'$hd3temp
#printf ' hd4temp:'$hd4temp
printf ' systemp:'$systemp

exit 0
```

<img src="/images/posts/qnap/p2.png">
